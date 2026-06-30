# Integration Architecture — OrthoNow CRM & Conversion Stack

**Version:** `v1.0.0`
**Scope:** Landing Page → HubSpot CRM → Karix WhatsApp → Google Ads Conversion

---

## Architecture Overview

```
[Landing Page]
      |
      | POST /api/v1/leads  (name, phone, clinic_preference)
      ↓
[Backend API  —  Node.js / Python]
      |
      |── 1. Validate & sanitise input
      |── 2. Search HubSpot by phone (deduplication)
      |── 3. Create or update HubSpot contact
      |── 4. Trigger Karix WhatsApp welcome message
      |── 5. Upload offline conversion to Google Ads
      |── 6. Return 200 OK to landing page
```

---

## Phone-Based Deduplication — Why It Matters

HubSpot's native deduplication key is **email address**. When two form
submissions share the same email, HubSpot automatically merges the records.

OrthoNow's landing page collects only **name, phone, and clinic preference
— no email address.** This means HubSpot's built-in deduplication is
completely blind to repeat submissions from the same patient.

Without explicit phone-based deduplication, a patient who submits the form
twice (e.g., once on mobile, once on desktop) will create two separate
contact records. This corrupts patient counts, splits conversation history,
and triggers duplicate WhatsApp messages — damaging the patient experience
and inflating pipeline numbers in HubSpot reports.

### Deduplication Flow

```
1. Receive form submission  →  phone: "9876543210"

2. Call HubSpot CRM Search API:
   POST https://api.hubapi.com/crm/v3/objects/contacts/search
   {
     "filterGroups": [{
       "filters": [{
         "propertyName": "phone",
         "operator":     "EQ",
         "value":        "9876543210"
       }]
     }],
     "properties": ["hs_object_id", "phone", "firstname", "clinic_preference"]
   }

3. If results.total > 0  →  UPDATE existing contact (PATCH)
   If results.total = 0  →  CREATE new contact (POST)
```

This single search call prevents duplicate records without requiring email,
and it is the only correct approach given OrthoNow's data collection scope.

### HubSpot Contact Properties Mapped

| Form Field | HubSpot Property |
|---|---|
| name (first word) | `firstname` |
| name (remaining) | `lastname` |
| phone | `phone` |
| clinic_preference | `clinic_preference` (custom property) |
| utm_source | `hs_analytics_source` |
| utm_campaign | `hs_analytics_last_touch_converting_campaign` |
| Submission timestamp | `notes_last_updated` |

---

## Karix WhatsApp Integration

Once the HubSpot contact is created or updated, the backend calls the
Karix WhatsApp Business API to send a templated confirmation message:

```
POST https://api.karix.io/whatsapp/v1/message
Authorization: Bearer <KARIX_API_KEY>
{
  "to":       "+919876543210",
  "template": {
    "name":     "appointment_confirmation",
    "language": { "code": "en" },
    "components": [{
      "type":       "body",
      "parameters": [
        { "type": "text", "text": "Rahul" },
        { "type": "text", "text": "Koramangala, Bengaluru" }
      ]
    }]
  }
}
```

**Template content (pre-approved by Meta):**
*"Hi {{1}}, your consultation request at OrthoNow {{2}} has been received.
Our team will call you within 2 hours to confirm your slot. — OrthoNow Team"*

The Karix call is made **after** HubSpot succeeds, so we never send a
WhatsApp message for a contact that wasn't saved.

---

## Google Ads Offline Conversion Upload

The backend uploads an offline conversion event to Google Ads using the
Google Ads API Offline Conversions endpoint. This is necessary because the
form submission originates on a landing page where the Google Click ID
(`gclid`) is available from the URL parameter:

```
Capture at submission:  gclid = getUrlParam('gclid')

Upload payload:
{
  "conversionName":   "Consultation Form Submitted",
  "conversionTime":   "2024-08-15T10:30:00+05:30",
  "gclid":            "TeSter-gclid-123",
  "conversionValue":  0.0,
  "currencyCode":     "INR"
}
```

This ties the form submission back to the exact ad click, enabling Smart
Bidding to optimise towards real leads.

---

## Failure Handling

**Retry:** All three downstream calls (HubSpot, Karix, Google Ads) are
wrapped in exponential backoff — up to 3 retries with delays of 1s, 2s,
and 4s. Transient HTTP 429 (rate limit) and 5xx errors are retried;
4xx validation errors are not.

**Queue:** If all retries fail, the job is pushed to a Redis-backed queue
(BullMQ or Celery). A background worker retries every 5 minutes for up to
24 hours before moving the job to the Dead Letter Queue.

**Dead Letter Queue (DLQ):** Failed jobs after 24 hours land in a dedicated
DLQ. An on-call engineer reviews and manually retriggers or escalates.

**Timeout Handling:** Each downstream API call has an explicit 8-second
timeout. A timed-out call is treated the same as a 5xx error and enters
the retry flow. The landing page receives a 200 response immediately — it
does not wait for downstream calls to complete (fire-and-acknowledge pattern).

**Logging:** Every API request and response (status code, latency, error
body) is written to a structured JSON log. Logs are shipped to Datadog /
CloudWatch with fields for `phone_hash`, `clinic`, `event_type`, and
`attempt_number`.

**Monitoring:** A Datadog monitor alerts if the HubSpot search error rate
exceeds 1% over a 5-minute window, or if queue depth exceeds 50 jobs.

**Alerting:** PagerDuty alert fires if: (a) the DLQ receives more than 10
jobs in 1 hour, or (b) the Karix WhatsApp delivery failure rate exceeds 5%.

**Fallback:** If HubSpot is unreachable for more than 2 consecutive minutes,
the backend writes submissions to a PostgreSQL `lead_fallback` table. A
scheduled job syncs these to HubSpot once the outage clears.

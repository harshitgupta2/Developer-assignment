# OrthoNow — Digital Growth Foundation

A production-ready technical foundation built before OrthoNow's Google Ads
campaigns go live. Covers GTM analytics architecture, a high-converting
consultation landing page, and a CRM integration design connecting HubSpot,
Karix WhatsApp, and Google Ads.

---

## Project Overview

**Client:** OrthoNow Healthcare Pvt. Ltd.
**Scope:** 9 orthopaedic clinics — Bengaluru, Hyderabad, Chennai
**Objective:** Fix a 2.1% conversion rate by establishing the correct
analytics and CRM plumbing before paid traffic is switched on.

The three tasks in this repo are designed to be implemented in order:
GTM schema first (so engineers know what to track), landing page second
(the conversion surface), integration third (so leads flow into HubSpot
and trigger WhatsApp messages automatically).

---

## Folder Structure

```
Developer-Assignment/
│
├── README.md                        ← This file
│
├── task-1/
│   ├── gtm-event-schema.md          ← Complete GTM event schema (18 events, 10 columns each)
│   └── booking-funnel.md            ← Booking funnel dataLayer JSON + GTM/GA4 config guide
│
├── task-2/
│   ├── index.html                   ← Self-contained landing page (open by double-click)
│   └── pagespeed-notes.md           ← PageSpeed optimisation rationale and score estimates
│
├── task-3/
│   └── integration-design.md        ← HubSpot + Karix + Google Ads architecture document
│
└── assets/                          ← Reserved for screenshots, diagrams, or exported assets
```

---

## How to Run

### Landing Page (Task 2)

No build step. No server. No npm install.

1. Navigate to `task-2/`
2. Double-click `index.html`
3. The page opens in your default browser

To test `dataLayer` events:
1. Open Chrome DevTools (F12)
2. Go to the **Console** tab
3. Fill in the form and click "Book My Free Consultation"
4. You will see `[OrthoNow Analytics] window.dataLayer:` logged with the
   full event object

To test call tracking:
1. Click any "Call Now" / "080 4709 4709" button
2. Check the Console for `call_now_click` event logs

### GTM Schema (Task 1)

Open `task-1/gtm-event-schema.md` in any Markdown viewer (VS Code Preview,
GitHub, Obsidian) to see the formatted table. The `booking-funnel.md` file
contains copy-pasteable JSON that goes directly into the booking widget's
frontend JavaScript.

### Integration Design (Task 3)

Open `task-3/integration-design.md` in any Markdown viewer. The architecture
diagram is text-based (no external image required).

---

## Tech Stack

| Layer | Technology | Rationale |
|---|---|---|
| Landing Page | Vanilla HTML5 / CSS3 / ES5 JS | Zero dependencies = zero PageSpeed penalties |
| CSS Architecture | CSS Custom Properties + `clamp()` | Token-based, mobile-first, no preprocessor needed |
| Icons | Inline SVG | No external requests; inherits CSS colour |
| Analytics | Google Tag Manager + GA4 | Industry standard; flexible tag management |
| CRM | HubSpot CRM Search API | Phone-based deduplication without email |
| Messaging | Karix WhatsApp Business API | WhatsApp is the dominant post-visit communication channel in India |
| Ads | Google Ads Offline Conversion Upload | Ties form submissions back to exact ad clicks for Smart Bidding |

---

## Features

**Task 1 — GTM Event Schema**
- 18 events covering the complete user journey
- 5-step booking funnel with actual JSON `dataLayer.push()` objects
- Call Now tracking across 3 page types (homepage, landing, clinic)
- WhatsApp widget tracking
- PDF download tracking
- Blog scroll depth at 5 thresholds (25 / 50 / 75 / 90 / 100%)
- GA4 Funnel Exploration configuration walkthrough
- Custom dimensions mapping table
- Consent Mode v2 notes

**Task 2 — Landing Page**
- Single HTML file — no build step, no dependencies
- Mobile-first CSS (320px → 375px → 768px → 1024px → desktop)
- Hero with headline targeting knee/back pain for ages 28–50
- Booking form with inline validation (Name, Phone, Clinic Preference)
- Thank-you state swap — no page reload
- `window.dataLayer` push on submit with 12 parameters
- All call buttons fire `call_now_click` dataLayer events
- Sticky mobile CTA bar with book + call actions
- Trust section: stats, specialties, features, clinic locations
- All-SVG icons, system font stack, zero external HTTP requests
- Estimated PageSpeed Mobile: 92–97
- Semantic HTML5 (`main`, `header`, `footer`, `section`, `nav`)
- ARIA labels, `role` attributes, `aria-live` for dynamic content

**Task 3 — Integration Design**
- Full architecture: Landing Page → Backend → HubSpot → Karix → Google Ads
- Phone-based deduplication via HubSpot CRM Search API (critical)
- Create vs Update decision logic
- Karix WhatsApp templated message with pre-approved template
- Google Ads offline conversion upload with `gclid`
- Failure handling: exponential backoff retry, Redis queue, DLQ,
  timeout handling, structured logging, Datadog monitoring, PagerDuty
  alerting, PostgreSQL fallback table

---

## Assignment Coverage

| PRD Requirement | Delivered | File |
|---|---|---|
| GTM Event Schema document | ✅ | `task-1/gtm-event-schema.md` |
| Booking funnel — Step 1 | ✅ | `task-1/booking-funnel.md` |
| Booking funnel — Step 2 | ✅ | `task-1/booking-funnel.md` |
| Booking funnel — Step 3 | ✅ | `task-1/booking-funnel.md` |
| Booking funnel — Completed | ✅ | `task-1/booking-funnel.md` |
| Booking funnel — Failed | ✅ | `task-1/booking-funnel.md` |
| Call Now events (3 page types) | ✅ | `task-1/gtm-event-schema.md` |
| WhatsApp Widget event | ✅ | `task-1/gtm-event-schema.md` |
| PDF Download event | ✅ | `task-1/gtm-event-schema.md` |
| Patient Guide Form event | ✅ | `task-1/gtm-event-schema.md` |
| Clinic Page View event | ✅ | `task-1/gtm-event-schema.md` |
| Blog Scroll 25% / 50% / 75% / 90% / 100% | ✅ | `task-1/gtm-event-schema.md` |
| GA4 Funnel Exploration setup | ✅ | `task-1/booking-funnel.md` |
| Single HTML landing page | ✅ | `task-2/index.html` |
| No frameworks / no CDN | ✅ | `task-2/index.html` |
| Mobile-first responsive design | ✅ | `task-2/index.html` |
| Hero with headline + form | ✅ | `task-2/index.html` |
| Fields: Name, Phone, Clinic Preference | ✅ | `task-2/index.html` |
| Trust section | ✅ | `task-2/index.html` |
| Sticky mobile CTA | ✅ | `task-2/index.html` |
| Thank-you state — no reload | ✅ | `task-2/index.html` |
| `consultation_form_submitted` dataLayer push | ✅ | `task-2/index.html` |
| `console.log(window.dataLayer)` | ✅ | `task-2/index.html` |
| PageSpeed ≥ 90 target | ✅ | `task-2/pagespeed-notes.md` |
| Integration architecture document | ✅ | `task-3/integration-design.md` |
| Phone-based HubSpot deduplication | ✅ | `task-3/integration-design.md` |
| HubSpot CRM Search API | ✅ | `task-3/integration-design.md` |
| Create vs Update logic | ✅ | `task-3/integration-design.md` |
| Karix WhatsApp integration | ✅ | `task-3/integration-design.md` |
| Google Ads conversion upload | ✅ | `task-3/integration-design.md` |
| Retry / Queue / DLQ / Timeout | ✅ | `task-3/integration-design.md` |
| Logging / Monitoring / Alerting / Fallback | ✅ | `task-3/integration-design.md` |

---

## Future Improvements

These are deliberate out-of-scope items — they are the logical next layer
once this foundation is live and generating data.

1. **A/B test the hero headline.** Run "Stop Living With Knee & Back Pain"
   against a benefit-led variant ("Walk Pain-Free in 6 Weeks"). GTM's
   built-in experiment support makes this straightforward with the existing
   schema.

2. **Add a `gclid` capture field.** A hidden `<input>` populated from
   `?gclid=` lets the backend associate every form submission with the
   exact Google Ads click — enabling true cost-per-lead tracking.

3. **Server-side GTM.** Move GA4, Google Ads, and HubSpot tags from the
   client to a server-side GTM container. This bypasses ad blockers,
   removes third-party scripts from the browser, and improves PageSpeed
   by another 5–10 points.

4. **Structured data (Schema.org).** Add `MedicalOrganization`,
   `LocalBusiness`, and `FAQPage` JSON-LD to the landing page. Improves
   organic rich results even though this is a paid landing page.

5. **Multi-language variants.** Kannada, Telugu, and Tamil variants of
   the landing page for hyper-local campaigns. The existing CSS token
   architecture requires no structural changes — only content differs.

6. **WhatsApp conversation handoff.** After the Karix template message,
   connect unresolved conversations to a HubSpot inbox via the WhatsApp
   Business Platform integration so the clinic team can reply in-CRM.

7. **GA4 → BigQuery export.** Export raw GA4 events to BigQuery for
   cross-channel attribution modelling once campaign data volume is
   sufficient for statistical significance.

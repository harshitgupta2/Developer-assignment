# Booking Funnel — dataLayer Implementation Guide

**Client:** OrthoNow Healthcare Pvt. Ltd.
**Scope:** Multi-step appointment booking widget
**Critical note:** GTM does not auto-detect form steps. Every push below
must be written explicitly in the frontend JavaScript.

---

## Funnel Architecture

```
Step 1                Step 2               Step 3              Confirmation
City + Specialty  →  Date + Time Slot  →  Personal Details  →  Booking ID
     ↓                    ↓                    ↓                    ↓
booking_step_         booking_step_        booking_step_       booking_
complete (1)          complete (2)         complete (3)        completed
                                                                    ↓
                                                               (on API error)
                                                             booking_failed
```

---

## Step 1 — Location & Specialty Selected

**Frontend action:** User selects a city from a dropdown and a specialty
from a list, then clicks the "Next" button to advance to Step 2.

**Where to fire:** Inside the `onClick` handler of the Step 1 "Next" button,
after validating that both `clinic_location` and `specialty` are non-empty.

```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Bengaluru",
  "specialty": "Knee Replacement",
  "device_type": "mobile",
  "traffic_source": "google_cpc",
  "utm_campaign": "orthonow-knee-blr-q3",
  "page_location": "https://orthonow.in/book-appointment",
  "page_title": "Book an Appointment — OrthoNow"
}
```

**Exact JavaScript:**
```javascript
document.getElementById('step1-next-btn').addEventListener('click', function () {
  var city      = document.getElementById('city-select').value;
  var specialty = document.getElementById('specialty-select').value;

  if (!city || !specialty) return; // validation runs before this push

  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push({
    event: 'booking_step_complete',
    step_number: 1,
    step_name: 'location_specialty_selected',
    clinic_location: city,
    specialty: specialty,
    device_type: getDeviceType(),         // 'mobile' | 'tablet' | 'desktop'
    traffic_source: getTrafficSource(),   // reads utm_source from URL params
    utm_campaign: getUrlParam('utm_campaign'),
    page_location: window.location.href,
    page_title: document.title
  });
});
```

**GTM trigger:**
- Trigger type: Custom Event
- Event name: `booking_step_complete`
- Filter: `{{DL - step_number}}` equals `1`
- Fires on: All Custom Events matching above

**GA4 event created:**
- Event name: `booking_step_complete`
- Parameters sent: all keys above as event parameters
- GA4 custom dimension `clinic_location` and `specialty` must be registered

**Funnel Exploration step:** Step 1 — "Location & Specialty Selected"
Filter condition: `event_name` equals `booking_step_complete`
AND custom dimension `step_number` equals `1`

---

## Step 2 — Date & Time Slot Selected

**Frontend action:** User picks an available date from a calendar picker
and selects a time slot (e.g., "10:00 AM – 10:30 AM"), then clicks "Next".

**Where to fire:** Inside the `onClick` handler of the Step 2 "Next" button,
after validating that `appointment_date` and `time_slot` are both set.

```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "date_time_selected",
  "clinic_location": "Bengaluru",
  "specialty": "Knee Replacement",
  "appointment_date": "2024-08-15",
  "appointment_time_slot": "10:00 AM",
  "clinic_branch": "Koramangala",
  "device_type": "mobile",
  "traffic_source": "google_cpc",
  "page_location": "https://orthonow.in/book-appointment"
}
```

**Exact JavaScript:**
```javascript
document.getElementById('step2-next-btn').addEventListener('click', function () {
  var date = document.getElementById('date-picker').value;      // "2024-08-15"
  var slot = document.getElementById('time-slot').value;        // "10:00 AM"
  var branch = document.getElementById('branch-select').value;  // "Koramangala"

  if (!date || !slot) return;

  window.dataLayer.push({
    event: 'booking_step_complete',
    step_number: 2,
    step_name: 'date_time_selected',
    clinic_location: window._bookingState.city,   // carried from Step 1
    specialty: window._bookingState.specialty,     // carried from Step 1
    appointment_date: date,
    appointment_time_slot: slot,
    clinic_branch: branch,
    device_type: getDeviceType(),
    traffic_source: getTrafficSource(),
    page_location: window.location.href
  });
});
```

**GTM trigger:**
- Trigger type: Custom Event
- Event name: `booking_step_complete`
- Filter: `{{DL - step_number}}` equals `2`

**GA4 event created:** `booking_step_complete` (same event name, step_number=2
differentiates it in Funnel Exploration)

**Funnel Exploration step:** Step 2 — "Date & Time Selected"

---

## Step 3 — Personal Details Entered

**Frontend action:** User enters their full name and mobile number in the
final form fields, then clicks "Confirm Booking".

**Where to fire:** Inside the `onClick` handler of the "Confirm Booking"
button, after client-side validation passes — but BEFORE the API call is made.
This ensures the event fires even if the API fails, allowing `booking_failed`
to be correctly attributed to Step 3.

```json
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "personal_details_entered",
  "clinic_location": "Bengaluru",
  "clinic_branch": "Koramangala",
  "specialty": "Knee Replacement",
  "appointment_date": "2024-08-15",
  "appointment_time_slot": "10:00 AM",
  "device_type": "mobile",
  "traffic_source": "google_cpc",
  "form_id": "booking_widget_step3",
  "page_location": "https://orthonow.in/book-appointment"
}
```

**Note:** Do NOT include the user's name or phone number in the dataLayer
push. PII must not flow into GA4 or Google Ads raw event data.

**Exact JavaScript:**
```javascript
document.getElementById('confirm-booking-btn').addEventListener('click', function (e) {
  e.preventDefault();

  var name  = document.getElementById('patient-name').value.trim();
  var phone = document.getElementById('patient-phone').value.trim();

  if (!name || !isValidPhone(phone)) return;

  // Step 3 dataLayer push fires BEFORE the API call
  window.dataLayer.push({
    event: 'booking_step_complete',
    step_number: 3,
    step_name: 'personal_details_entered',
    clinic_location: window._bookingState.city,
    clinic_branch: window._bookingState.branch,
    specialty: window._bookingState.specialty,
    appointment_date: window._bookingState.date,
    appointment_time_slot: window._bookingState.timeSlot,
    device_type: getDeviceType(),
    traffic_source: getTrafficSource(),
    form_id: 'booking_widget_step3',
    page_location: window.location.href
  });

  // Now call the booking API
  submitBooking(name, phone);
});
```

**GTM trigger:**
- Trigger type: Custom Event
- Event name: `booking_step_complete`
- Filter: `{{DL - step_number}}` equals `3`

**GA4 event created:** `booking_step_complete` (step_number=3)

**Funnel Exploration step:** Step 3 — "Personal Details Entered"

---

## Booking Completed

**Frontend action:** The backend booking API returns a `200 OK` response
with a `booking_id`. The confirmation screen is shown to the user.

**Where to fire:** Inside the `.then()` callback of the booking API fetch,
after `booking_id` is extracted from the response.

```json
{
  "event": "booking_completed",
  "booking_id": "ON-BLR-20240815-4829",
  "clinic_location": "Bengaluru",
  "clinic_branch": "Koramangala",
  "specialty": "Knee Replacement",
  "appointment_date": "2024-08-15",
  "appointment_time_slot": "10:00 AM",
  "device_type": "mobile",
  "traffic_source": "google_cpc",
  "utm_campaign": "orthonow-knee-blr-q3",
  "utm_medium": "cpc",
  "utm_source": "google",
  "page_location": "https://orthonow.in/book-appointment",
  "conversion_value": 0,
  "currency": "INR"
}
```

**Exact JavaScript:**
```javascript
function submitBooking(name, phone) {
  fetch('/api/v1/appointments', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      name: name,
      phone: phone,
      city: window._bookingState.city,
      branch: window._bookingState.branch,
      specialty: window._bookingState.specialty,
      date: window._bookingState.date,
      time_slot: window._bookingState.timeSlot
    })
  })
  .then(function (res) {
    if (!res.ok) throw new Error('API_ERROR:' + res.status);
    return res.json();
  })
  .then(function (data) {
    window.dataLayer.push({
      event: 'booking_completed',
      booking_id: data.booking_id,
      clinic_location: window._bookingState.city,
      clinic_branch: window._bookingState.branch,
      specialty: window._bookingState.specialty,
      appointment_date: window._bookingState.date,
      appointment_time_slot: window._bookingState.timeSlot,
      device_type: getDeviceType(),
      traffic_source: getTrafficSource(),
      utm_campaign: getUrlParam('utm_campaign'),
      utm_medium: getUrlParam('utm_medium'),
      utm_source: getUrlParam('utm_source'),
      page_location: window.location.href,
      conversion_value: 0,
      currency: 'INR'
    });

    showConfirmationScreen(data.booking_id);
  })
  .catch(function (err) {
    handleBookingFailure(err.message);
  });
}
```

**GTM trigger:**
- Trigger type: Custom Event
- Event name: `booking_completed`
- No additional filters needed

**GA4 event created:** `booking_completed`
- Mark as a conversion in GA4 Admin → Events → Mark as conversion

**Google Ads conversion:**
- Create a conversion action in Google Ads: "Appointment Booked"
- In GTM, fire a Google Ads Conversion tag triggered by `booking_completed`
- Pass `{{DL - booking_id}}` as the transaction ID to prevent duplicate counting

**Funnel Exploration step:** Step 4 — "Booking Confirmed"

---

## Booking Failed

**Frontend action:** The API returns a non-2xx status, times out, or returns
a `slot_unavailable` error code. The user sees an error message.

**Where to fire:** Inside the `.catch()` block of the booking API fetch,
and also inside the `.then()` block if the API returns a business-logic
error (e.g., `{ "error": "slot_unavailable" }`).

```json
{
  "event": "booking_failed",
  "error_type": "slot_unavailable",
  "error_message": "Selected slot is no longer available. Please choose another time.",
  "error_code": "SLOT_TAKEN",
  "clinic_location": "Bengaluru",
  "clinic_branch": "Koramangala",
  "specialty": "Knee Replacement",
  "appointment_date": "2024-08-15",
  "appointment_time_slot": "10:00 AM",
  "step_at_failure": 3,
  "device_type": "mobile",
  "traffic_source": "google_cpc",
  "page_location": "https://orthonow.in/book-appointment"
}
```

**Exact JavaScript:**
```javascript
function handleBookingFailure(errorMessage) {
  var errorType = 'unknown_error';
  var errorCode = 'UNKNOWN';

  if (errorMessage.includes('SLOT_TAKEN') || errorMessage.includes('slot_unavailable')) {
    errorType = 'slot_unavailable';
    errorCode = 'SLOT_TAKEN';
  } else if (errorMessage.includes('timeout') || errorMessage.includes('408')) {
    errorType = 'timeout';
    errorCode = 'REQUEST_TIMEOUT';
  } else if (errorMessage.includes('API_ERROR')) {
    errorType = 'api_error';
    errorCode = errorMessage.replace('API_ERROR:', '').trim();
  }

  window.dataLayer.push({
    event: 'booking_failed',
    error_type: errorType,
    error_message: errorMessage,
    error_code: errorCode,
    clinic_location: window._bookingState.city,
    clinic_branch: window._bookingState.branch,
    specialty: window._bookingState.specialty,
    appointment_date: window._bookingState.date,
    appointment_time_slot: window._bookingState.timeSlot,
    step_at_failure: 3,
    device_type: getDeviceType(),
    traffic_source: getTrafficSource(),
    page_location: window.location.href
  });

  showErrorMessage(errorType);
}
```

**GTM trigger:**
- Trigger type: Custom Event
- Event name: `booking_failed`

**GA4 event created:** `booking_failed`
- Do NOT mark as a conversion
- Use in custom explorations to identify failure patterns by `error_type`,
  `clinic_location`, and `specialty`

**Funnel Exploration:** This event is used as a drop-off identifier.
Users who fired `booking_step_complete` (step 3) but NOT `booking_completed`
represent the failed/abandoned cohort.

---

## GA4 Funnel Exploration — Configuration Steps

1. Open GA4 → **Explore** → **Blank** → change Technique to **Funnel Exploration**
2. Set the date range to cover your campaign period
3. Add the following steps in order:

| Step | Condition |
|------|-----------|
| Step 1 | Event name equals `booking_step_complete` AND `step_number` equals `1` |
| Step 2 | Event name equals `booking_step_complete` AND `step_number` equals `2` |
| Step 3 | Event name equals `booking_step_complete` AND `step_number` equals `3` |
| Step 4 | Event name equals `booking_completed` |

4. Enable **"Make steps consecutive"** — this enforces the funnel order
5. Set **Counting method** to `Once per session`
6. Add **Breakdown dimension:** `clinic_location` to see drop-off by city
7. Add **Breakdown dimension:** `specialty` to see which specialties have the highest drop-off
8. Add **Segment:** `device_type = mobile` vs `device_type = desktop` to identify device-based friction

### Reading the Funnel

- High drop between Step 1 → Step 2: The date picker UX has friction or slot availability is poor
- High drop between Step 2 → Step 3: Price/commitment anxiety — consider adding social proof before personal details
- High drop between Step 3 → confirmation: API issues or form validation errors — investigate `booking_failed` events

---

## Shared Utility Functions

These functions are referenced across all dataLayer pushes above.
Place them at the top of the booking widget's JavaScript file.

```javascript
function getDeviceType() {
  var w = window.innerWidth;
  if (w < 768)  return 'mobile';
  if (w < 1024) return 'tablet';
  return 'desktop';
}

function getUrlParam(key) {
  return new URLSearchParams(window.location.search).get(key) || '';
}

function getTrafficSource() {
  return getUrlParam('utm_source') || 'direct';
}

function isValidPhone(phone) {
  var clean = phone.replace(/[\s\-\(\)]/g, '');
  return /^(\+91)?[6-9]\d{9}$/.test(clean);
}
```

# GTM Event Schema — OrthoNow Digital Growth Foundation

**Version:** `v1.0.0`
**Client:** OrthoNow Healthcare Pvt. Ltd.
**Scope:** Bengaluru · Hyderabad · Chennai (9 clinics)
**Prepared for:** Google Tag Manager + GA4 + Google Ads implementation

---

## How to Read This Schema

Every row in the tables below represents one GTM tag configuration. The
**dataLayer Event** column is the exact string the frontend pushes — GTM
listens for it via a Custom Event trigger. The **Parameters** column lists
every variable that must be present in the `dataLayer.push()` call for
the corresponding GA4 tag to fire correctly.

**Implementation rule:** GTM does NOT automatically detect multi-step form
interactions. The frontend JavaScript must push each event explicitly.

---

## 1. Appointment Booking Funnel

> These five events are the most business-critical. Every event must fire
> for GA4 Funnel Exploration to visualise drop-off between steps.

| # | Event Name | Description | Trigger Type | Trigger Condition | dataLayer Event | Parameters | GA4 Event Name | GA4 Report | Audience Usage | Google Ads Usage |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | `booking_step_complete` | User completes Step 1: selects city and specialty from the booking widget | Custom Event | `event` equals `booking_step_complete` AND `step_number` equals `1` | `booking_step_complete` | `step_number` (1), `step_name` ("location_specialty_selected"), `clinic_location` ("Bengaluru / Hyderabad / Chennai"), `specialty` ("Knee Replacement / Back Pain / etc."), `device_type`, `traffic_source`, `page_location` | `booking_step_complete` | Funnel Exploration — Step 1 | "Booking Started" remarketing list — users who began but didn't complete | Bid modifier +15% for users who reached Step 1 |
| 2 | `booking_step_complete` | User completes Step 2: selects preferred date and available time slot | Custom Event | `event` equals `booking_step_complete` AND `step_number` equals `2` | `booking_step_complete` | `step_number` (2), `step_name` ("date_time_selected"), `appointment_date`, `appointment_time_slot`, `clinic_location`, `specialty`, `device_type`, `traffic_source` | `booking_step_complete` | Funnel Exploration — Step 2 | "Date Selected" segment — high-intent users for RLSA | RLSA target — increase bid by 25% for users at Step 2 |
| 3 | `booking_step_complete` | User completes Step 3: enters personal details (name and phone number) | Custom Event | `event` equals `booking_step_complete` AND `step_number` equals `3` | `booking_step_complete` | `step_number` (3), `step_name` ("personal_details_entered"), `clinic_location`, `specialty`, `appointment_date`, `device_type`, `traffic_source`, `form_id` | `booking_step_complete` | Funnel Exploration — Step 3 | "Details Entered" audience — near-conversion for aggressive remarketing | Smart Bidding signal — very high-intent conversion signal |
| 4 | `booking_completed` | Booking is confirmed: backend returns success, confirmation number shown to user | Custom Event | `event` equals `booking_completed` | `booking_completed` | `booking_id`, `clinic_location`, `specialty`, `appointment_date`, `appointment_time_slot`, `device_type`, `traffic_source`, `page_location` | `booking_completed` | Conversions Report, Funnel Exploration — Step 4 | "Converted Patients" suppression list — exclude from acquisition campaigns | Primary conversion action for Smart Bidding (Target CPA / Max Conversions) |
| 5 | `booking_failed` | Booking submission fails: API error, timeout, or slot no longer available | Custom Event | `event` equals `booking_failed` | `booking_failed` | `error_type` ("api_error / timeout / slot_unavailable"), `error_message`, `clinic_location`, `specialty`, `step_at_failure`, `device_type`, `traffic_source` | `booking_failed` | Custom Exploration — Error Analysis | "Failed Booking" list — retarget with reassurance messaging | Negative signal — exclude from conversion count; use for ad copy testing |

---

## 2. Call Now

> Three separate entries because `button_location` determines which
> Google Ads campaign gets credit. Homepage callers have different intent
> than landing page callers arriving from paid ads.

| # | Event Name | Description | Trigger Type | Trigger Condition | dataLayer Event | Parameters | GA4 Event Name | GA4 Report | Audience Usage | Google Ads Usage |
|---|---|---|---|---|---|---|---|---|---|---|
| 6 | `call_now_click` | User clicks any phone number or "Call Now" button on the main homepage | Custom Event | `event` equals `call_now_click` AND `button_location` equals `homepage` | `call_now_click` | `button_location` ("homepage"), `phone_number`, `page_location`, `device_type`, `traffic_source`, `page_title` | `call_now_click` | Events Report, User Explorer | "Homepage Callers" audience — high organic intent | Secondary conversion — supports Smart Bidding without inflating primary |
| 7 | `call_now_click` | User clicks call button on the paid consultation landing page | Custom Event | `event` equals `call_now_click` AND `button_location` contains `landing` | `call_now_click` | `button_location` ("header / sticky_bar / cta_section"), `phone_number`, `page_location`, `device_type`, `traffic_source`, `utm_campaign` | `call_now_click` | Events Report, Acquisition Report | "Landing Page Callers" — paid traffic segment for RLSA | Import as offline conversion via Google Ads call conversion tracking |
| 8 | `call_now_click` | User clicks call button on any individual clinic location page | Custom Event | `event` equals `call_now_click` AND `button_location` equals `clinic_page` | `call_now_click` | `button_location` ("clinic_page"), `clinic_name`, `clinic_city`, `phone_number`, `device_type`, `traffic_source`, `page_location` | `call_now_click` | Events Report, User Explorer | "Clinic Callers" segmented by city — local remarketing | Location extension optimisation — match caller city to nearest clinic ad |

---

## 3. WhatsApp Widget

| # | Event Name | Description | Trigger Type | Trigger Condition | dataLayer Event | Parameters | GA4 Event Name | GA4 Report | Audience Usage | Google Ads Usage |
|---|---|---|---|---|---|---|---|---|---|---|
| 9 | `whatsapp_widget_click` | User clicks the floating WhatsApp chat widget | Custom Event | `event` equals `whatsapp_widget_click` | `whatsapp_widget_click` | `widget_location` ("floating_button / inline_cta"), `page_type` ("homepage / landing / clinic / blog"), `device_type`, `traffic_source`, `page_location` | `whatsapp_widget_click` | Events Report | "WhatsApp Engagers" — warm audience preferring async communication | Secondary conversion — used in Display/YouTube remarketing campaigns |

---

## 4. PDF Download

| # | Event Name | Description | Trigger Type | Trigger Condition | dataLayer Event | Parameters | GA4 Event Name | GA4 Report | Audience Usage | Google Ads Usage |
|---|---|---|---|---|---|---|---|---|---|---|
| 10 | `pdf_download` | User clicks to download any PDF asset (patient guide, procedure overview, pricing sheet) | Custom Event | `event` equals `pdf_download` | `pdf_download` | `file_name` (e.g., "knee-replacement-patient-guide.pdf"), `file_category` ("patient_guide / procedure_info / pricing"), `page_location`, `device_type`, `traffic_source`, `page_title` | `file_download` | Events Report, Content Report | "PDF Downloaders" — research-phase users for nurture campaigns | Secondary conversion — supports content-driven ad targeting |

---

## 5. Patient Guide Form

| # | Event Name | Description | Trigger Type | Trigger Condition | dataLayer Event | Parameters | GA4 Event Name | GA4 Report | Audience Usage | Google Ads Usage |
|---|---|---|---|---|---|---|---|---|---|---|
| 11 | `patient_guide_form_submit` | User submits the inline patient guide request form (name + phone to receive a PDF via WhatsApp) | Custom Event | `event` equals `patient_guide_form_submit` | `patient_guide_form_submit` | `form_id` ("patient_guide_form"), `guide_topic` ("knee / back / hip / sports"), `clinic_preference`, `device_type`, `traffic_source`, `page_location` | `generate_lead` | Events Report, Conversions | "Guide Requesters" — top-of-funnel leads for email/WhatsApp nurture | Micro-conversion for Lead Gen campaigns; feed into Customer Match |

---

## 6. Clinic Page Views

| # | Event Name | Description | Trigger Type | Trigger Condition | dataLayer Event | Parameters | GA4 Event Name | GA4 Report | Audience Usage | Google Ads Usage |
|---|---|---|---|---|---|---|---|---|---|---|
| 12 | `clinic_page_view` | User views any individual clinic detail page (location, doctors, timings) | Page View | URL path matches `/clinics/*` | `clinic_page_view` | `clinic_name` ("Koramangala / Banjara Hills / etc."), `clinic_city` ("Bengaluru / Hyderabad / Chennai"), `page_location`, `device_type`, `traffic_source`, `page_title` | `page_view` (with custom dimension `clinic_name`) | Pages and Screens, User Explorer | "Clinic Page Visitors" segmented by city — location-specific remarketing | Local campaign targeting — match viewed clinic city to ad location |

---

## 7. Blog Scroll Depth

> Five separate entries — one per scroll threshold. GTM's native Scroll
> Depth trigger handles these, but the frontend must NOT suppress default
> scroll behaviour. Each threshold fires only once per page session.

| # | Event Name | Description | Trigger Type | Trigger Condition | dataLayer Event | Parameters | GA4 Event Name | GA4 Report | Audience Usage | Google Ads Usage |
|---|---|---|---|---|---|---|---|---|---|---|
| 13 | `blog_scroll_depth` | User scrolls to 25% of a blog article | Scroll Depth | Vertical scroll depth ≥ 25% on `/blog/*` pages | `blog_scroll_depth` | `scroll_percentage` (25), `page_location`, `blog_title`, `blog_category` ("knee / back / sports / general"), `device_type`, `traffic_source` | `scroll` | Engagement Report | "Blog Browsers" (25%) — wide awareness audience | Awareness-stage Display remarketing |
| 14 | `blog_scroll_depth` | User scrolls to 50% of a blog article | Scroll Depth | Vertical scroll depth ≥ 50% on `/blog/*` pages | `blog_scroll_depth` | `scroll_percentage` (50), `page_location`, `blog_title`, `blog_category`, `device_type`, `traffic_source` | `scroll` | Engagement Report | "Blog Readers" (50%) — qualified awareness audience | Consideration-stage remarketing with educational ad copy |
| 15 | `blog_scroll_depth` | User scrolls to 75% of a blog article | Scroll Depth | Vertical scroll depth ≥ 75% on `/blog/*` pages | `blog_scroll_depth` | `scroll_percentage` (75), `page_location`, `blog_title`, `blog_category`, `time_on_page`, `device_type`, `traffic_source` | `scroll` | Engagement Report | "Engaged Readers" (75%) — high-intent research audience | RLSA with +10% bid modifier; serve "Book a Consultation" CTA ads |
| 16 | `blog_scroll_depth` | User scrolls to 90% of a blog article | Scroll Depth | Vertical scroll depth ≥ 90% on `/blog/*` pages | `blog_scroll_depth` | `scroll_percentage` (90), `page_location`, `blog_title`, `blog_category`, `time_on_page`, `device_type`, `traffic_source` | `scroll` | Engagement Report | "Deep Readers" (90%) — very high engagement, likely evaluating treatment | RLSA with +20% bid modifier; use in remarketing for procedure-specific ads |
| 17 | `blog_scroll_depth` | User reaches the end of a blog article (100%) | Scroll Depth | Vertical scroll depth = 100% on `/blog/*` pages | `blog_scroll_depth` | `scroll_percentage` (100), `page_location`, `blog_title`, `blog_category`, `time_on_page`, `read_completion` ("true"), `device_type`, `traffic_source` | `scroll` | Engagement Report | "Article Completers" — highest-intent content audience | Target for next-day remarketing with direct conversion CTA |

---

## 8. Landing Page Form Submission

| # | Event Name | Description | Trigger Type | Trigger Condition | dataLayer Event | Parameters | GA4 Event Name | GA4 Report | Audience Usage | Google Ads Usage |
|---|---|---|---|---|---|---|---|---|---|---|
| 18 | `consultation_form_submitted` | User submits the consultation booking form on the paid landing page | Custom Event | `event` equals `consultation_form_submitted` | `consultation_form_submitted` | `form_id` ("hero_consultation_form"), `clinic_preference`, `page_type` ("landing"), `device_type`, `traffic_source`, `utm_campaign`, `page_location`, `page_title` | `generate_lead` | Conversions Report | "Form Submitters" — primary converted audience; suppress from acquisition | Primary conversion action for all Google Ads campaigns; feeds Smart Bidding |

---

## GTM Container Configuration Notes

### Custom Dimensions Required in GA4

| Dimension Name | Scope | Populated By |
|---|---|---|
| `clinic_location` | Event | `clinic_location` dataLayer variable |
| `specialty` | Event | `specialty` dataLayer variable |
| `device_type` | Event | `device_type` dataLayer variable |
| `traffic_source` | Session | `traffic_source` dataLayer variable |
| `page_type` | Event | `page_type` dataLayer variable |
| `blog_category` | Event | `blog_category` dataLayer variable |
| `button_location` | Event | `button_location` dataLayer variable |
| `step_number` | Event | `step_number` dataLayer variable |

### GTM Variable Configuration

All parameters above must be mapped as **Data Layer Variables** in GTM
before the corresponding GA4 Event tags can read them. Variable names
must exactly match the dataLayer key names in this document.

### Consent Mode

All tags must be configured to respect Google Consent Mode v2. Analytics
tags should fire in `analytics_storage: 'granted'` state only. Conversion
tags require both `analytics_storage` and `ad_storage` granted.

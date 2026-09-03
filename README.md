# Upstage Track — Working Prototype

## What this actually is

A **fully functional, click-through web app** (`index.html`) covering the entire employee
and admin workflow you specified, running against a simulated cloud database
(your browser's local storage standing in for a real backend). Open the file in
any phone or desktop browser — no install, no server required.

**Real, working features in this build:**
- Live camera capture (`getUserMedia`) for Time In / Time Out selfies and job photos
- Real GPS capture (`navigator.geolocation`) at every attendance event, with reverse
  geocoding to a readable address (falls back to raw coordinates if offline)
- Geofence distance-check logic (Upstage Workshop / Accor Stadium zones included)
- Break rules engine — Morning Tea (Smoko) 20 min **paid**, Lunch 30 min **unpaid**,
  configurable — with automatic gross/net hours calculation
- Full employee flow: Login → Select Job → Time In (photo+GPS) → Work → Start/End
  Break → Time Out (photo+GPS) → Work Summary → Timesheet
- Full admin flow: Live dashboard, employee management (create/disable/reset PIN),
  job management, attendance record detail (photo+time+location+job in one screen),
  manual entry with mandatory reason, timesheet approve/reject, edit-after-approval
  with forced audit trail, filterable reports with **working CSV export**, audit log,
  company/attendance/break/notification settings, Xero integration screen
- Demo data for Upstage Co and all 7 employees / 5 jobs you specified

## What is deliberately simulated (and why)

Some things in your spec need infrastructure this environment cannot provide —
a real cloud server, an Apple/Google developer account, or a live third-party
API credential. Rather than fake these silently, here's exactly what's simulated
and what a production build needs:

| Area | In this prototype | What production needs |
|---|---|---|
| Backend/database | Browser local storage | A real cloud DB (e.g. Postgres via Supabase/Firebase/AWS) + API layer |
| Native iOS/Android app | Mobile-web app (installable to home screen, works offline-first) | Wrap in React Native / Flutter, or ship as a PWA; needs Apple Developer ($99/yr) + Google Play ($25 one-off) accounts to publish |
| Xero integration | UI/config screens only, "Connect" is simulated | Real Xero OAuth 2.0 app (Client ID/Secret from developer.xero.com), server-side token exchange, calls to Xero's Timesheets API |
| Push notifications | Not implemented (needs a backend + APNs/FCM) | Firebase Cloud Messaging / Apple Push Notification service |
| Background geofence tracking | Not implemented (spec says off by default anyway) | Native background location APIs, iOS/Android permission flows |
| Server-authoritative timestamp | Uses device clock | A real backend stamps the time on arrival, not the device |
| PDF export | Placeholder message | Server-side rendering (e.g. Puppeteer) or a PDF library |
| Multi-user real-time sync | Single-browser only | A real backend + websockets/polling so admin sees live updates across devices |

**Nothing here is faked to look real** — the Xero and PDF screens explicitly tell
you, in the UI, what's simulated versus what needs real credentials.

## Test accounts (PIN-based, no email needed for demo)

- **Employee role** (PIN `1111`): Alex Rivera, Shadelle Nguyen, Chris Doyle, Bodhi Marsh, Joel Kaine
- **Manager role** (PIN `1111`): Julia Santos — sees Dashboard, Employees (her team), Jobs, Live, Timesheets, Reports, Alerts. No Settings/Xero/Audit/Payroll.
- **Payroll role** (PIN `1111`): Will Foster — sees Dashboard, Reports, Payroll export, Alerts only.
- **Super Admin** (PIN `9999`): Rosa Alvear — full access to every screen.

Each account routes straight to the right interface for its permission level — Employee-level
accounts land on the clock in/out home screen; everyone else lands in the admin console with
sidebar/tabs filtered to what their role is allowed to see.

## What's new in this pass

- **Device + GPS accuracy capture** at every Clock In/Out, shown on the confirmation screen and in the record detail
- **Task categories** at clock-in — Warehouse/Prep, Site/Bump In, Pack Down, Workshop, Office/Admin — so hours roll up by activity type, not just by job
- **Employee record fields expanded**: Employee ID, Position, Department, Employment type, Hourly rate (visible to Admin/Super Admin/Payroll only), Manager, Default location, Permission level
- **Roles & permissions**: Employee, Manager, Payroll, Admin, Super Admin, each with a distinct set of visible admin tabs
- **Real Draft → Submitted → Approved / Rejected / Corrected timesheet pipeline**: employees explicitly submit completed shifts; admins approve or reject with a required reason; editing an approved record marks it "Corrected" instead of silently staying "Approved"
- **Employee-initiated correction requests**: an employee can request a fix to a submitted or approved shift (e.g. forgot to clock out); admins see a dedicated Correction Requests queue and can approve (auto-applies the new time, logs original vs. corrected) or reject
- **Expanded Reports** with a report-type selector: Attendance, Job Labour (with $ cost using each employee's hourly rate), Department, Late Arrivals, Missing Clock Out, Overtime, Location Exceptions — all still filterable and CSV-exportable
- **Payroll Export screen**: Employee ID / Employee / Regular / OT / Total, restricted to Payroll/Admin/Super Admin, CSV export, and it only ever includes Approved/Corrected timesheets
- **Notifications/Alerts screen**: auto-generated from real data — missing clock-outs, overtime, late arrivals, timesheets awaiting approval, rejected timesheets

## What's still simulated (unchanged from before)

See the table below — the backend, native app wrapper, Xero OAuth, and push notifications
still require real infrastructure this environment can't provide. Nothing new in this pass
changes that list.

## Test data

Company: **Upstage Co**. Jobs: ATEEZ Tour, Foo Fighters Australia, Workshop, Accor
Stadium — Rigging, General Office. Several days of attendance, breaks, and work logs
are pre-loaded so History/Timesheet/Reports/Audit Log all have content immediately.

## Try the full workflow end to end

1. Log in as an employee → tap **TIME IN** → pick a job → take a live selfie → watch
   GPS get captured and reverse-geocoded.
2. Tap **START BREAK** (choose Morning Tea or Lunch) → **END BREAK**.
3. Tap **TIME OUT** → selfie → write a work summary → see hours calculated.
4. Log out, log in as admin (Julia Santos) → **Timesheets** tab → open the new
   pending entry → see photo + time + location + job in one screen → **Approve**.
5. **Reports** tab → filter by employee/job/date → **Export CSV** (downloads a real file).
6. **Xero** tab → read the integration note explaining what's real vs simulated.

## Database schema (conceptual — this is what a real backend would implement)

`Employees`, `Admins`, `Jobs`, `JobAssignments`, `Attendance`, `Breaks`, `WorkLogs`,
`Photos` (as blob storage references, not embedded), `Locations`, `Timesheets`,
`TimesheetApprovals`, `XeroMappings`, `XeroSyncLogs`, `Notifications`, `AuditLogs`,
`CompanySettings` — each with a unique ID, foreign keys as described in your spec,
and row-level access rules so employees only ever see their own records.

## Credentials/config a real deployment would need from you

- A Xero developer app (Client ID + Secret) and your company's Xero organisation connected via OAuth
- A cloud database + hosting account (e.g. Supabase, Firebase, or AWS)
- Apple Developer Program and Google Play Console accounts, if publishing as native apps
- A maps/geocoding API key (Google Maps or Mapbox) for production-grade reverse geocoding and map display
- Push notification setup (Firebase Cloud Messaging covers both platforms)

## Known limitations of this prototype specifically

- All data lives in one browser's local storage — clearing browser data or switching
  devices resets it. Real usage needs the shared backend described above.
- Photos are stored as base64 images in local storage, which is fine for a demo but
  not how a production app should store media (should use object storage like S3).
- The "server timestamp" is your device's clock, since there's no server here.
- Reverse geocoding depends on internet access to a free public API (OpenStreetMap
  Nominatim); production should use a paid geocoding API with an SLA.

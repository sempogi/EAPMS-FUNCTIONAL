# FI EAPMS Technical Change Notes

**Document version:** 1.2.1  
**System checkpoint:** Android 1.12.1 build 17 / Local EAPMS Gate 2  
**Prepared:** 2026-10-10  
**Current status:** **WORKING - FURTHER LIVE TESTING REQUIRED**

---

## GitHub Description

### Short repository description

> FI EAPMS is an integrated Android and PHP/MySQL employee attendance platform with office-geofenced time punches, offline-first synchronization, local Timekeeper processing, leave workflows, Firebase notifications, digital employee ID, GPS activity logs, and bulletin publishing.

### topics

```text
employee-attendance
attendance-system
timekeeping
maui
android
php
mysql
firebase
firebase-cloud-messaging
geofencing
offline-first
qr-code
data-protection
```

### About text

```text
Android and PHP/MySQL employee attendance, leave, bulletin, and notification platform with office-geofenced offline Time In/Time Out synchronization.
```

---

## 1. Current Working Checkpoint

The following modules are working and retained as the current checkpoint:

- Android employee registration and device installation identity.
- Digital employee ID and QR display with Android device-credential protection.
- Office configuration with multiple authorized geofence anchors.
- GPS Activity Log with local storage and online submission.
- Online leave request submission and status tracking.
- Android manager endorsement and rejection workflow.
- Local-manager approval workflow.
- Local Timekeeper/Admin processing workflow for Android-endorsed leave.
- Firebase Cloud Messaging device registration and notifications.
- Rejection notification reasons and notification icon support.
- Android bulletin reader and authorized bulletin editor.
- Local EAPMS bulletin editor and Firebase broadcast.
- Dedicated Android OFFICE-only Time In and Time Out.
- Offline-first local attendance-punch storage.
- Online attendance staging with `UPLOADED` status.
- Local EAPMS Timekeeper/Admin attendance processing.
- Online attendance finalization with `ACCEPTED` status.
- Mock-location warning and permanent `MOCK` source marking.

The full attendance route has been observed working. Additional controlled tests remain recommended for offline retries, duplicate protection, mock classification, failed online acknowledgements, and multi-employee use.

---

## 2. Android Version History

### Version 1.11.0

Major working checkpoint for:

- Firebase Cloud Messaging.
- Leave endorsement and rejection notifications.
- Notification icon and rejection-reason display.
- Bulletin reader and bulletin editor.
- Bulletin publisher authorization.
- Local EAPMS bulletin publishing integration.

### Version 1.12.0, build 16

Introduced Gate 1 Android attendance punches:

- Added dedicated `TIME IN` and `TIME OUT` buttons to the Activity Log page.
- Added `SYNC ATTENDANCE`.
- Enforced one Time In and one Time Out per employee per local date.
- Required Time In before Time Out.
- Required a fresh GPS reading.
- Required GPS accuracy of 100 meters or better.
- Allowed attendance only inside a configured authorized office anchor.
- Rejected `REMOTE`, unavailable GPS, inaccurate GPS, and unmatched geofence results.
- Added separate private monthly attendance JSON storage.
- Added offline-first `PENDING` records.
- Added automatic upload when connectivity is available.
- Added manual and page-open synchronization.
- Added Android reconciliation from `UPLOADED` to `ACCEPTED`.

### Version 1.12.1, build 17

Introduced mock-location classification:

- Android reads the MAUI location mock-provider indicator.
- Mock detection does not permanently lock the application.
- Mock detection does not automatically accuse the employee of misconduct.
- Time In, Time Out, and Save GPS Activity show an extreme warning.
- The user may cancel without consuming the daily action.
- Continuing requires explicit acknowledgement.
- Continued records are permanently marked `MOCK`.
- The mock classification remains visible through `PENDING`, `UPLOADED`, and `ACCEPTED`.
- Genuine records are marked `GENUINE`.

---

## 3. Attendance Punch Architecture

### 3.1 Attendance lifecycle

```text
Android capture
    -> local PENDING
    -> online UPLOADED
    -> local EAPMS FETCHED
    -> local attendance/event transaction
    -> online ACCEPTED
    -> Android ACCEPTED reconciliation
```

### 3.2 Android business rules

```text
TIME IN
- Maximum once per employee per local date.
- Requires fresh GPS.
- Requires an authorized OFFICE geofence match.
- May be saved offline.

TIME OUT
- Maximum once per employee per local date.
- Requires an existing TIME IN.
- Requires a new fresh GPS check.
- Requires an authorized OFFICE geofence match.
- May be saved offline.
```

### 3.3 Disallowed conditions

No attendance punch is created under these conditions:

- Phone is outside all authorized office anchors.
- GPS is unavailable.
- Location permission is unavailable.
- GPS accuracy exceeds the configured 100-meter limit.
- No recognized office anchor matches.
- The daily action already exists.
- Time Out is attempted without Time In.
- A mock warning is displayed and the user selects Cancel.

### 3.4 Separate local Android storage

Attendance punches are not mixed with GPS Activity Log records.

```text
App-private storage:
attendance_punches/attendance_yyyy_MM.json
```

Local statuses:

```text
PENDING   Saved locally and waiting for online upload.
UPLOADED  Stored by the online attendance API.
ACCEPTED  Processed successfully by local EAPMS and acknowledged online.
```

### 3.5 Automatic upload behavior

When the phone is online:

```text
Valid punch
-> save locally first
-> immediately attempt upload
-> mark local record UPLOADED after server success
```

When the phone is offline:

```text
Valid punch
-> save locally as PENDING
-> retain record safely
-> retry when Activity Log opens or SYNC ATTENDANCE is tapped
```

No permanent Android background service is required. Synchronization is event-driven.

---

## 4. Mock-Location Handling

### 4.1 Policy

The application records the technical source classification without making a disciplinary conclusion.

```text
GENUINE  Android did not identify the captured fix as mock.
MOCK     Android identified the captured fix as mock or simulated.
```

### 4.2 Mock warning behavior

For Time In, Time Out, or Save GPS Activity:

```text
Mock location detected
-> show SIMULATED LOCATION DETECTED warning
-> explain permanent MOCK marking
-> allow Cancel
-> allow Continue and Save as Mock
```

Cancel behavior:

```text
No local record
No online upload
No daily action consumed
```

Continue behavior:

```text
location_source = MOCK
mock_location_detected = true
mock_warning_acknowledged = true
mock_warning_acknowledged_at = ISO-8601 acknowledgement time
```

### 4.3 Server consistency checks

The online attendance upload endpoint accepts a mock record only if:

```text
location_source = MOCK
mock_location_detected = true
mock_warning_acknowledged = true
mock_warning_acknowledged_at is present
```

Incomplete mock acknowledgements are rejected with:

```text
MOCK_WARNING_NOT_ACKNOWLEDGED
```

Inconsistent source metadata is rejected with:

```text
INVALID_LOCATION_SOURCE
```

### 4.4 Limitation

Mock-location marking is a practical source-integrity indicator, not conclusive evidence of employee intent. Advanced rooted, hooked, or modified Android environments may interfere with platform indicators. Preserve the marker as audit evidence and leave administrative conclusions to authorized reviewers.

---

## 5. Online Attendance APIs

### 5.1 Android submission API

```text
Endpoint:
/staff/verify/attendance_punch_api.php

Partner:
Android Services/AttendancePunchApiService.cs

Credential:
Dedicated Android attendance-upload token

Purpose:
- Submit new punches.
- Return duplicate/idempotent results.
- Return online status for Android reconciliation.
- Validate OFFICE-only evidence.
- Validate GPS accuracy and office radius.
- Validate daily action uniqueness.
- Validate mock-location acknowledgement metadata.
```

### 5.2 Local EAPMS feed API

```text
Endpoint:
/staff/verify/attendance_punch_feed_api.php

Partner:
Local includes/attendance_punch_sync.php

Credential:
Dedicated local attendance-processing token

Purpose:
Return online records whose status is UPLOADED.
```

### 5.3 Local EAPMS status API

```text
Endpoint:
/staff/verify/attendance_punch_status_api.php

Partner:
Local includes/attendance_punch_sync.php

Credential:
Dedicated local attendance-processing token

Purpose:
Change UPLOADED to ACCEPTED only after successful local processing.
```

### 5.4 Shared attendance API helper

```text
File:
/staff/verify/attendance_punch_common.php

Purpose:
- Processing-token validation.
- Shared JSON opening and locking.
- Shared JSON response format.
- Shared attendance storage path.
```

### 5.5 Online storage

```text

```

The JSON file is outside the public web root and is updated with file locking.

---

## 6. Local EAPMS Attendance Gate 2

### 6.1 New local page

```text
attendance_processing.php
```

Menu label:

```text
Online Attendance
```

Authorized roles:

```text
TIMEKEEPER
ADMINISTRATOR
```

### 6.2 Local supporting files

```text
config/attendance_punch_online.php
includes/attendance_punch_sync.php
```

### 6.3 Local staging table

```text
online_attendance_punches
```

Primary responsibilities:

- Preserve immutable `record_id`.
- Preserve unique `server_reference`.
- Match the employee number to the local employee record.
- Store the raw online payload.
- Track fetch, local processing, acknowledgement, and error states.
- Link the staging record to local attendance and attendance-event records.

### 6.4 Local queue statuses

```text
FETCHED
EMPLOYEE_NOT_FOUND
LOCAL_SAVED_PENDING_ACK
ACCEPTED
```

`LOCAL_SAVED_PENDING_ACK` means the local attendance transaction succeeded but the online `ACCEPTED` callback failed. The user receives a `Retry Acceptance` action that does not insert attendance again.

### 6.5 Processing requirements

```text
1. Retrieve UPLOADED records.
2. Process TIME IN before TIME OUT.
3. Validate employee existence.
4. Check record and server-reference duplication.
5. Begin local database transaction.
6. Insert or update the daily attendance row.
7. Insert the attendance event.
8. Recalculate attendance values.
9. Commit the transaction.
10. Call the online status endpoint.
11. Mark online and local staging records ACCEPTED.
```

### 6.6 Calendar and attendance updates

The original Android punch timestamp is used, not the Timekeeper's processing time.

After processing, the central attendance calculator updates applicable values such as:

- Attendance mode.
- Attendance status.
- First Time In.
- Last Time Out.
- Late minutes.
- Worked hours.
- Overtime hours.
- Holiday and rest-day results where applicable.

---

## 7. Leave Processing

### 7.1 Local-manager route

```text
Online PENDING
-> copied locally
-> local manager approves
-> local attendance calendar updated
-> online PENDING changes directly to APPROVED
```

### 7.2 Android-manager and Timekeeper route

```text
Online PENDING
-> Android manager endorses
-> online FOR_PROCESSING
-> local EAPMS imports request
-> Timekeeper/Admin processes it
-> local calendar updated
-> online FOR_PROCESSING changes to APPROVED
```

### 7.3 Rejection route

Android manager rejection includes:

- Rejection reason.
- Firebase notification.
- Notification icon support.
- Online status update.

### 7.4 Supported online leave transitions

```text
PENDING -> CANCELLED
PENDING -> APPROVED
FOR_PROCESSING -> APPROVED
```

---

## 8. Firebase Cloud Messaging

Completed behavior includes:

- Android installation registration.
- Device-token update support.
- Manager endorsement notification.
- Manager rejection notification.
- Rejection-reason display.
- Notification icon support.
- Bulletin broadcast to active registered devices.
- Notification routing to the appropriate application page.

Firebase credentials and service-account material must remain outside source control.

---

## 9. Bulletin Module

### 9.1 Android bulletin module

- Bulletin reader.
- Authorized editor.
- Publisher authorization using employee and installation identity.
- Priority support: `NORMAL`, `IMPORTANT`, and `URGENT`.
- Optional expiration.
- Firebase broadcast after publication.

### 9.2 Local bulletin editor

```text
Local EAPMS bulletin_editor.php
-> bulletin_local_publish_api.php
-> secure bulletins.json
-> Firebase broadcast
```

The improved UI includes:

- Larger announcement body area.
- Balanced editor proportions.
- Desktop two-column editor and preview.
- Responsive mobile layout.
- Live preview.
- Live character counters.
- Priority and expiration controls.

---

## 10. Endpoint and Partner Inventory

### Employee and device

```text
employee API
<-> Android employee registration/service

device registration API
<-> Android device registration service

FCM registration API
<-> Android Firebase token registration service

office configuration API
<-> Android office configuration service
```

### Leave

```text
leave submission API
<-> Android LeaveApiService

manager leave API
<-> Android ManagerLeaveService

leave feed/status APIs
<-> Local EAPMS leave relay and processing files

leave_status_update_api.php
<-> Local manager approval and Timekeeper/Admin finalization
```

### Bulletin

```text
bulletin_api.php
<-> Android bulletin reader/editor

bulletin_local_publish_api.php
<-> Local EAPMS bulletin_editor.php

fcm_sender.php
<-> Bulletin and workflow notification broadcasts
```

### Attendance punch

```text
attendance_punch_api.php
<-> Android AttendancePunchApiService

attendance_punch_feed_api.php
<-> Local attendance punch fetch process

attendance_punch_status_api.php
<-> Local attendance acceptance callback

attendance_punch_common.php
<-> Shared online attendance processing helper
```

---

## 11. Token Separation and Warnings

Never combine the tokens for these partner groups:

```text
Android attendance upload
Local EAPMS attendance processing
GPS Activity Log upload
Leave submission
Leave manager action
Local leave status update
Android bulletin publishing
Local bulletin publishing
Firebase registration
Device registration
```

### Required operational rules

- Rotate one token family without changing unrelated modules.
- Keep token labels and API partnerships documented.
- Never copy production token values into public GitHub repositories.
- Never include Firebase service-account JSON in source control.
- Prefer server environment variables or protected configuration outside the public web root for future hardening.
- Preserve the current matching tokens during live testing unless both partner files are updated together.

### Token migration recommendation

After live testing stabilizes:

```text
1. Inventory all endpoint/partner token pairs.
2. Generate new random values per pair.
3. Move online secrets outside public files where hosting permits.
4. Update one pair at a time.
5. Test the pair before rotating the next credential.
6. Revoke obsolete credentials.
7. Record the rotation date without recording the secret value.
```

---

## 12. Setup and Deployment Notes

### Android

- Current target: `.NET 10 Android`.
- Current application version: `1.12.1`.
- Current Android build number: `17`.
- Build using the complete latest project, not isolated fragments from an older release.

### Online attendance files

```text
attendance_punch_api.php
attendance_punch_common.php
attendance_punch_feed_api.php
attendance_punch_status_api.php
```

### Local EAPMS

- Run `setup.php` only when required for installation or schema update.
- Manually delete `setup.php` after successful setup.
- The installer function collision was corrected by separating the setup-specific table-existence function from the runtime database helper.

Correct function separation:

```text
setup.php: eapms_setup_table_exists(PDO $pdo, string $tableName)
config/database.php: eapms_table_exists(string $tableName)
```

---

## 13. Known Testing Requirements

The system is marked **WORKING - FURTHER LIVE TESTING REQUIRED**.

Recommended remaining tests:

### Android attendance

- Genuine GPS Time In inside each configured office anchor.
- Genuine GPS Time Out inside each configured office anchor.
- Outside-fence rejection.
- GPS-disabled result.
- Poor-accuracy result.
- Duplicate Time In prevention.
- Duplicate Time Out prevention.
- Time Out before Time In prevention.
- Offline Time In or Time Out followed by later synchronization.
- Application restart while a record is `PENDING`.
- Duplicate upload retry using the same immutable record ID.

### Mock location

- Warning appears for mock Time In.
- Cancel produces no record.
- Continue produces permanent `MOCK` metadata.
- Mock GPS Activity Log is visibly marked.
- Server rejects incomplete mock acknowledgement.
- Mock marker remains after `UPLOADED`.
- Mock marker remains after `ACCEPTED`.

### Gate 2

- Retrieve one uploaded Time In.
- Retrieve one uploaded Time Out.
- Process Time In before Time Out.
- Verify local attendance event creation.
- Verify calendar calculation.
- Verify online `ACCEPTED` update.
- Simulate acknowledgement failure and test `Retry Acceptance`.
- Confirm retry does not duplicate attendance or attendance events.
- Confirm repeated feed retrieval reports records as already local.
- Confirm missing employee records are isolated as `EMPLOYEE_NOT_FOUND`.

### Multi-user and concurrency

- Two employees uploading simultaneously.
- Two local users retrieving the queue.
- Repeated Process clicks.
- File-lock behavior during simultaneous online requests.

---

## 14. Recommended Backup Set

Before every live rollout, preserve:

```text
Android project source
Signed APK or last known working APK
Local EAPMS project folder
Local MySQL database dump
Online attendance_punches.json
Online leave JSON files
Online bulletins.json
Online device/FCM registration files
Office anchor configuration
Protected token/configuration files
Firebase service-account configuration
```

Do not store production secrets in a public backup repository.

---

## 15. Architectural Summary

The current FI EAPMS design keeps authority layered:

```text
Android
- captures employee-controlled actions
- saves locally first
- uploads evidence
- does not finalize authoritative attendance

Online server
- stages uploaded requests and records
- enforces duplicate and transition rules
- provides controlled API channels

Local EAPMS
- remains the final authoritative processor
- validates employee records
- updates attendance and calendar data
- acknowledges successful processing online

Timekeeper/Admin
- retains explicit final processing authority
```

This preserves the governing principle:

> Data may be exposed or distributed through controlled channels, but authority and access remain centralized, layered, and explicitly controlled.

---

## 16. Final Checkpoint

```text
Checkpoint date: 2026-10-10
Android: 1.12.1 build 17
Local attendance Gate 2: installed and working
Online attendance staging: working
Automatic online upload: working
Online-to-local retrieval: working
Local processing and ACCEPTED callback: working
Mock-location warning and source marking: working
Overall status: WORKING - FURTHER LIVE TESTING REQUIRED
```

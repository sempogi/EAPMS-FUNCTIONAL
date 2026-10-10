# FI EAPMS

FI EAPMS is an integrated employee attendance, leave, bulletin, and notification platform built with .NET MAUI for Android and PHP/MySQL for local administration.

The platform supports office-based attendance, offline-first mobile synchronization, controlled leave processing, employee bulletins, Firebase notifications, digital employee identification, and centralized Timekeeper or Administrator review.

> **Project status:** Working checkpoint. Additional live and multi-user testing is ongoing.

## Core Features

### Android Employee Application

- Employee registration and device association
- Digital employee ID and QR display
- Device-credential protection for sensitive screens
- Office-location configuration using multiple geofence anchors
- GPS activity logging
- Dedicated Time In and Time Out controls
- One Time In and one Time Out per employee per day
- Time Out available only after Time In
- Offline-first local storage
- Automatic synchronization when connectivity is available
- Manual attendance synchronization
- Attendance state display using Pending, Uploaded, and Accepted statuses
- Leave request submission and status tracking
- Manager leave endorsement and rejection
- Bulletin reader and authorized bulletin editor
- Firebase push notifications

### Attendance Processing

The attendance workflow separates mobile capture from final administrative authority:

```text
Android capture
-> saved locally
-> uploaded when online
-> retrieved by local EAPMS
-> processed by Timekeeper or Administrator
-> attendance calendar updated
-> final status returned to Android
```

Attendance punches require:

- A fresh GPS position
- Acceptable GPS accuracy
- A matching authorized office geofence
- A valid employee registration
- Compliance with daily Time In and Time Out rules

Remote attendance punches are not allowed.

### Offline-First Synchronization

Attendance punches are saved on the device before an upload is attempted.

```text
Pending
-> stored locally and waiting for upload

Uploaded
-> received by the online staging service

Accepted
-> finalized by the local EAPMS
```

If connectivity is unavailable, the record remains pending and can synchronize later without losing the original attendance timestamp.

### Mock-Location Classification

When Android identifies a location as mock or simulated, the application displays a prominent warning before saving.

The user may cancel without creating a record. If the user continues, the record is permanently classified as:

```text
MOCK LOCATION
```

The classification remains visible throughout synchronization and administrative processing. It records the technical location source without automatically determining intent.

### Local EAPMS Administration

The local PHP/MySQL application provides:

- Employee attendance management
- Attendance calendar review
- Timekeeper and Administrator processing
- Leave processing and calendar integration
- Employee input mapping support
- Holiday and leave-credit management
- Attendance reports and exports
- Online attendance retrieval
- Duplicate-processing protection
- Retry support when final online acknowledgement fails
- Bulletin publishing
- Role-based access control

### Leave Workflow

The system supports two controlled approval paths:

```text
Online request
-> local manager approval
-> local calendar update
-> final online approval
```

```text
Online request
-> Android manager endorsement
-> local Timekeeper or Administrator processing
-> local calendar update
-> final online approval
```

Manager rejection supports recorded reasons and employee notification.

### Bulletin and Notification Module

- Android bulletin reader
- Authorized Android bulletin editor
- Local bulletin editor
- Announcement priority levels
- Optional expiration dates
- Firebase broadcast notifications
- Notification routing to the relevant application page

## Architecture

```text
Android application
-> employee interaction, local-first capture, and synchronization

Online service
-> protected staging, controlled APIs, and notification delivery

Local EAPMS
-> authoritative processing, attendance database, and calendar updates

Timekeeper or Administrator
-> final review and processing authority
```

The architecture follows a centralized-authority model: information may move through controlled channels, while final authority remains with approved local roles.

## Technology Stack

### Mobile

- .NET MAUI
- C#
- Android
- Firebase Cloud Messaging

### Local Administration

- PHP
- MySQL
- HTML, CSS, and JavaScript

### Integration

- JSON-based synchronization
- Token-protected API partnerships
- Offline-first mobile storage
- File locking for online staging records
- Role-based administrative processing

## Security and Privacy Principles

- Separate credentials for unrelated integration channels
- No production credentials committed to source control
- Firebase service credentials excluded from the repository
- Sensitive online storage kept outside the public web root
- Explicit role checks for administrative processing
- Duplicate and replay protection for synchronized records
- Neutral technical classification of mock-location records
- Location collection only during employee-initiated GPS actions

## Current Version

```text
Android application: 1.12.1
Android build: 17
Status: Working, with further live testing in progress
```

## Testing Priorities

- Genuine office Time In and Time Out
- Outside-geofence rejection
- Offline capture and later synchronization
- Duplicate-action prevention
- Mock-location warning and classification
- Multi-user synchronization
- Failed acknowledgement recovery
- Attendance calendar recalculation
- Concurrent online staging requests



## Description

```text
Android and PHP/MySQL employee attendance, leave, bulletin, and notification platform with office-geofenced offline Time In/Time Out synchronization.
```

## Topics

```text
employee-attendance
attendance-system
timekeeping
dotnet-maui
android
php
mysql
firebase
geofencing
offline-first
qr-code
role-based-access-control
```
## Download

Download the latest available Android build from Google Drive:

- [Download FI EAPMS for Android](https://drive.google.com/file/d/1TqV2pEgVo7wxLvrtoxB60kI-HyVKAZJ_/view?usp=sharing)

> The linked build is provided for controlled testing. Verify the source, version, and release notes before installation.


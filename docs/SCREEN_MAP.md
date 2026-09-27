# Medica Web App — V1 Screen Map

This document defines the V1 screen structure and navigation of the Medica Web App.

It is intentionally concise. Detailed behavior and business rules belong in `PRODUCT_SPEC.md`.

---

# 1. Entry Point

```text
Login
├── Volunteer login
└── Admin login
```

The same login surface may technically handle both account types, but users are routed to the appropriate interface after authentication.

On first login with a temporary password:

```text
Login
└── Change Password Required
    └── Continue to appropriate Home/Dashboard
```

---

# 2. Volunteer Application

Main navigation:

```text
Volunteer
├── Home
├── Events
└── Profile
```

---

## 2.1 Volunteer Home

```text
Home
├── Next Event
│   ├── Event summary
│   ├── RSVP
│   └── View Event Details
│
└── Total Volunteering Hours
```

Possible states:

```text
Home
├── Upcoming event available
├── No upcoming events
└── Next relevant event cancelled
```

---

## 2.2 Volunteer Events

```text
Events
├── Upcoming
│   └── Event Details
│
└── Past
    └── Event Details
```

Upcoming events are listed nearest first.

Past events are listed newest first.

---

## 2.3 Volunteer Event Details

```text
Event Details
├── Event information
├── Description
├── Attendance count
│
├── Upcoming event
│   └── RSVP controls
│
└── Past event
    └── Personal attendance result
```

Possible attendance result:

```text
Attended
└── Actual volunteering hours

Did not attend

Cancelled
```

---

## 2.4 Volunteer Profile

```text
Profile
├── Profile picture
├── Personal information
│   ├── Name
│   ├── Username
│   ├── School
│   ├── Year of birth
│   └── Volunteer since
│
├── Total volunteering hours
│   └── View Volunteering History
│
└── Account
    ├── Change password
    └── Change profile picture
```

School is editable.

Other protected profile information is read-only for volunteers.

---

## 2.5 Volunteer History

```text
Volunteering History
├── Total volunteering hours
└── Attended events
    ├── Event name
    ├── Date
    └── Credited hours
```

Only attended events appear here.

---

# 3. Admin Application

Main navigation:

```text
Admin
├── Dashboard
├── Events
├── Volunteers
└── Admins
```

Attendance is accessed through Events rather than existing as a separate main navigation item.

---

# 4. Admin Dashboard

```text
Dashboard
├── Next Event
│   ├── Manage Event
│   └── Attendance
│
├── Volunteers
│   ├── View Volunteers
│   └── Add Volunteer
│
└── Events
    ├── View All Events
    └── Create Event
```

No analytics, charts, leaderboards, or statistics are required in V1.

---

# 5. Admin Volunteer Management

```text
Volunteers
├── Active Volunteers
│   └── Volunteer Details
│
├── Show Inactive Volunteers
│   └── Volunteer Details
│
└── Add Volunteer
```

---

## 5.1 Add Volunteer

```text
Add Volunteer
└── Create Volunteer
    └── Credentials Created
        ├── Copy credentials
        ├── View Volunteer
        └── Back to Volunteers
```

The credentials screen is shown after successful account creation.

---

## 5.2 Volunteer Details

```text
Volunteer Details
├── Personal information
│   └── Edit information
│
├── Total volunteering hours
│   └── View Volunteering History
│
└── Account actions
    ├── Reset password
    ├── Deactivate volunteer
    └── Reactivate volunteer
```

For inactive volunteers:

```text
Volunteer Details
└── Reactivate Volunteer
```

---

# 6. Admin Events

```text
Events
├── Create Event
│
├── Upcoming Events
│   ├── Manage Event
│   └── Attendance
│
└── Past Events
    ├── View / Manage Event
    └── Attendance
```

Cancelled events remain visible where relevant.

---

# 7. Create Event

```text
Create Event
├── Event information form
└── Create Event
    └── Manage Event
```

After successful creation, the admin is taken to the event management screen.

---

# 8. Manage Event

```text
Manage Event
├── Event information
├── Edit Event
├── Attendance
│
└── Event Actions
    ├── Cancel Event
    ├── Restore Event
    └── Delete Event
```

Available actions depend on event state and existing attendance records.

---

## 8.1 Edit Event

```text
Edit Event
└── Save Changes
    └── Manage Event
```

The same core fields are used as the Create Event form.

---

# 9. Attendance

```text
Attendance
├── Event summary
├── Attendance recorded counter
├── Add Unexpected Attendee
│
└── Attendance Table
    ├── Coming
    ├── Not Coming
    └── No Response
```

Each attendance row contains:

```text
Volunteer
├── Attendance status
├── Actual hours
└── Admin-only comment
```

Attendance statuses:

```text
Not Recorded
Attended
Absent
```

Changes are saved immediately.

---

## 9.1 Unexpected Attendee Flow

```text
Attendance
└── Add Unexpected Attendee
    ├── Search active volunteers
    └── Select volunteer
        └── Attendance row
```

If the selected volunteer is already present in the attendance table, no duplicate record is created.

---

# 10. Admin Accounts

```text
Admins
├── Active Admins
│   └── Admin Details
│
└── Add Admin
```

Inactive admins may also remain accessible for reactivation.

---

## 10.1 Add Admin

```text
Add Admin
└── Create Admin
    └── Credentials Created
        ├── Copy credentials
        ├── View Admin
        └── Back to Admins
```

---

## 10.2 Admin Details

```text
Admin Details
├── Account information
│   └── Edit information
│
└── Account actions
    ├── Reset password
    ├── Deactivate admin
    └── Reactivate admin
```

The last active admin cannot be deactivated.

An admin cannot deactivate their own account while logged in.

---

# 11. Authentication Flows

## 11.1 Normal Login

```text
Login
└── Authenticate
    ├── Volunteer → Volunteer Home
    └── Admin → Admin Dashboard
```

---

## 11.2 First Login

```text
Login with temporary password
└── Change Password Required
    └── Save new password
        ├── Volunteer → Volunteer Home
        └── Admin → Admin Dashboard
```

---

## 11.3 Password Reset

Volunteer:

```text
Volunteer contacts admin
└── Admin resets password
    └── Temporary password generated
        └── Volunteer logs in
            └── Change Password Required
```

Admin:

```text
Admin resets another admin's password
└── Temporary password generated
    └── Admin logs in
        └── Change Password Required
```

---

# 12. Navigation Summary

## Volunteer

```text
Home
Events
└── Event Details

Profile
└── Volunteering History
```

## Admin

```text
Dashboard

Volunteers
├── Add Volunteer
├── Volunteer Details
└── Volunteering History

Events
├── Create Event
├── Manage Event
├── Edit Event
└── Attendance

Admins
├── Add Admin
└── Admin Details
```

---

# 13. V1 Navigation Principle

Navigation should remain shallow.

Most tasks should require no more than one or two screen transitions from the main navigation.

The application should avoid unnecessary nested settings, dashboards, tabs, or secondary navigation.

The priority is fast access to:

- The next event
- RSVP
- Attendance
- Volunteer management
- Event management
- Volunteering history

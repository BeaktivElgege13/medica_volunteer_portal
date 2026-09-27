# Medica Web App — V1 Product Specification

## 1. Product Overview

The Medica Web App is a private web application for managing Medica volunteers, events, RSVPs, attendance, and volunteering hours.

The application is intended for a relatively small organization with approximately 30–40 active volunteers. It should remain simple, mobile-friendly, and easy to use.

The primary purpose of the system is:

> Organize Medica events, know who plans to attend, record who actually attended, and maintain accurate volunteer-hour records.

The application is not intended to be a social network, communication platform, or public-facing volunteer portal.

Medica already uses other communication channels such as WhatsApp and Viber for urgent communication, so messaging and notification systems are not part of V1.

---

# 2. User Types

There are two separate account types:

- Volunteer
- Admin

These roles are operationally separate.

An admin is not a volunteer and does not participate in events, RSVP, or accumulate volunteering hours.

A volunteer cannot be promoted into an admin account. If a volunteer later becomes part of Medica leadership, a separate admin account should be created for that person.

---

# 3. Volunteer Accounts

## 3.1 Account Creation

Volunteers cannot register themselves.

Volunteer accounts are created manually by an admin.

When creating a volunteer, the admin provides:

- First name
- Last name
- Year of birth
- School
- Volunteering since year

Example:

- First name: Amina
- Last name: Hadžić
- Year of birth: 2008
- School: Prva gimnazija Zenica
- Volunteering since: 2024

Only the year is stored for `Volunteering since`.

The school field may be left empty if it is not applicable.

Volunteer accounts are not permanently deleted in V1. Former volunteers are deactivated instead.

---

## 3.2 Username Generation

Volunteer usernames are generated automatically.

The username should generally be created from:

- First letter of the first name
- Followed by the surname

Bosnian characters should be normalized.

Examples:

- č / ć → c
- š → s
- ž → z
- đ → d

Example:

Amina Hadžić → `ahadzic`

If the username already exists, a numeric suffix is added.

Examples:

- `ahadzic`
- `ahadzic1`
- `ahadzic2`

Volunteer usernames must be unique.

Once assigned, the username does not automatically change if the volunteer's name is later edited.

Admins do not manually select volunteer usernames in V1.

---

## 3.3 Temporary Password

When a volunteer account is created, the system generates a temporary password.

The admin is shown:

- Username
- Temporary password

The admin can copy these credentials and provide them to the volunteer.

The temporary password should only be shown at the time it is generated.

The volunteer must choose a new password on their first login.

If a new temporary password is generated later, any previous temporary password must immediately stop working.

---

## 3.4 Password Reset

If a volunteer forgets their password, they contact an admin.

The admin can generate a new temporary password.

The volunteer must choose a new password the next time they log in.

There is no email-based or self-service password reset system in V1.

---

## 3.5 Volunteer Permissions

Volunteers can edit:

- Profile picture
- School
- Password

Volunteers cannot edit:

- First name
- Last name
- Username
- Year of birth
- Volunteering since year
- Account status
- Volunteering hours directly

These protected values can only be changed by admins.

---

## 3.6 Volunteer Data Validation

The system should prevent obviously invalid profile values.

Examples:

- Year of birth must be a plausible year
- Volunteering since year cannot be later than the current year
- Volunteering since year should not be earlier than the volunteer's birth year

Exact validation boundaries may be implemented conservatively rather than using overly restrictive assumptions.

---

# 4. Inactive Volunteers

Former volunteers must not be deleted from the system.

Instead, they are marked as inactive.

An inactive volunteer:

- Cannot log in
- Cannot RSVP
- Cannot change existing RSVPs
- Cannot participate in new event workflows

Historical data must remain preserved, including:

- Profile information
- Attendance records
- Volunteering hours
- Historical event participation
- Existing historical RSVP records

Deactivating a volunteer must not erase or alter their existing RSVP, attendance, or hour records.

Inactive volunteers remain accessible to admins.

They may later be reactivated if necessary.

Reactivation restores account access but does not modify historical RSVP, attendance, or volunteering-hour records.

Inactive volunteers should not appear in the normal active-volunteer list. Admins can access them through a separate `Show inactive volunteers` area.

Inactive volunteers must still appear by name in historical event and attendance records.

---

# 5. Admin Accounts

Admins are Medica leaders who manage the system.

Initially, the system may contain only one admin.

Existing admins can create additional admin accounts.

All admins have the same permissions in V1.

There are no separate admin levels or permission tiers.

---

## 5.1 Admin Account Creation

When creating an admin, the creator provides:

- First name
- Last name
- Username

The admin username may be entered manually because the number of admin accounts will be very small.

Admin usernames must be unique.

The system generates a temporary password.

The new admin must change the password after their first login.

---

## 5.2 Admin Password Reset

Admins can reset another admin's password.

The system generates a temporary password and requires the affected admin to change it on their next login.

Generating a new temporary password immediately invalidates the previous password or temporary password being replaced.

---

## 5.3 Admin Deactivation

Admins can be deactivated.

A deactivated admin can no longer access the admin portal.

The system must prevent the last active admin account from being deactivated.

This protection must apply to any future account-removal functionality as well.

An admin should also not be able to deactivate their own account while logged in. Another admin must perform that action.

Inactive admins may later be reactivated.

---

# 6. Volunteer Navigation

The volunteer-facing application contains three main areas:

- Home
- Events
- Profile

The volunteer experience should remain intentionally simple.

---

# 7. Volunteer Home

The Home page displays:

1. The next upcoming event
2. Total volunteering time

No analytics, charts, event statistics, or yearly breakdowns are shown.

---

## 7.1 Next Event

The next event section displays:

- Event name
- Date
- Start and end time
- Location
- Expected volunteering time
- Number of volunteers attending
- Volunteer RSVP controls
- RSVP deadline
- Link to full event details

Example:

```text
NEXT EVENT

Humanitarian Bazaar

17 October 2026
10:00 – 16:00

Gradski trg

Expected volunteering time: 6 hours

18 volunteers attending

Your response:

[ I'M COMING ]   [ I CAN'T COME ]

RSVP by: 16 October 2026 at 10:00

[ View event details ]
```

Volunteer names are never shown in the attendance count.

Only the number of confirmed volunteers is visible.

---

## 7.2 No Upcoming Events

If there are no upcoming events:

```text
There are currently no upcoming events.
```

---

## 7.3 Cancelled Next Event

If the next event is cancelled, the cancellation must be displayed prominently.

A cancelled event may remain visible on Home until its originally scheduled end time plus the event buffer defined by the system.

After that point, Home should move to the next relevant upcoming event.

Volunteers cannot change RSVP responses while an event is cancelled.

---

## 7.4 Total Volunteering Time

The bottom of Home displays the volunteer's total accumulated volunteering time.

Example:

```text
81

VOLUNTEERING HOURS
```

There is no yearly breakdown in V1.

---

# 8. Volunteer Events Page

The Events page uses a chronological list rather than a calendar.

It contains two sections:

- Upcoming
- Past

There is no search or filtering in V1.

---

## 8.1 Upcoming Events

Upcoming events are sorted nearest first.

Each item may display:

- Event date
- Event name
- Time
- Location
- Expected volunteering time
- Number attending
- Volunteer RSVP state

Possible RSVP states include:

- Attending
- Not attending
- No response
- RSVP closed

The event list is primarily an overview.

Opening an event leads to the Event Details page.

---

## 8.2 Past Events

The Past section displays all past Medica events, not only events attended by the volunteer.

Past events are sorted newest first.

Each event displays:

- Date
- Event name
- Attendance result

Possible states include:

- Attended
- Did not attend
- Cancelled

Cancelled events must not appear as `Did not attend`.

---

# 9. Volunteer Event Details

The Event Details page is shared between upcoming and past events.

The displayed content changes based on the event state.

---

## 9.1 Upcoming Event Details

Display:

- Event name
- Date
- Start and end time
- Location
- Expected volunteering time
- Number of volunteers attending
- Description
- RSVP controls
- RSVP deadline

Example:

```text
HUMANITARIAN BAZAAR

17 October 2026
10:00 – 16:00

Gradski trg

Expected volunteering time
6 hours

18 volunteers attending

DESCRIPTION

Event description...

YOUR RSVP

[ I'M COMING ]   [ I CAN'T COME ]

You can change your response until
16 October 2026 at 10:00.
```

If the event is cancelled, RSVP controls are disabled.

---

## 9.2 Past Event Details

For past events, RSVP controls disappear.

Instead, the volunteer sees their attendance information.

If attended:

```text
YOUR ATTENDANCE

Attended
5.5 volunteering hours
```

If not attended:

```text
YOUR ATTENDANCE

Did not attend
```

Admin-only attendance comments are never visible to volunteers.

---

# 10. Volunteer Profile

The Profile page displays:

- Profile picture
- First and last name
- Username
- School
- Year of birth
- Volunteering since year
- Total volunteering time
- Account actions

Example:

```text
MY PROFILE

[ Profile photo ]

Amina Hadžić

PERSONAL INFORMATION

Username
ahadzic

School
Prva gimnazija Zenica
[ Edit ]

Year of birth
2008

Volunteer since
2024

VOLUNTEERING

Total volunteering time

81 hours

[ View volunteering history ]

ACCOUNT

[ Change password ]
[ Change profile picture ]
```

If no profile picture exists, the UI should show a default avatar or initials rather than a broken or empty image.

---

# 11. Volunteer Volunteering History

Volunteering history is a separate page accessed from the Profile page.

It contains only events the volunteer actually attended.

Events that the volunteer missed are not shown here.

History is sorted newest first.

Each record displays:

- Event date
- Event name
- Actual volunteering hours credited

Example:

```text
VOLUNTEERING HISTORY

Total volunteering time
81 hours

17 OCT 2026
Humanitarian Bazaar
5 hours

04 SEP 2026
Charity Workshop
6 hours

18 JUN 2026
Fundraiser
4 hours
```

There is no yearly total or grouping requirement in V1.

---

# 12. Admin Navigation

The admin interface contains:

- Dashboard
- Events
- Volunteers
- Admins

Attendance is accessed through events rather than as a separate top-level navigation item.

---

# 13. Admin Dashboard

The Admin Dashboard should remain simple and action-focused.

It contains:

- Next event
- Volunteer shortcuts
- Event shortcuts

No charts, analytics, yearly statistics, recent activity feeds, or leaderboards are shown in V1.

---

## 13.1 Next Event

The main dashboard card displays:

- Event name
- Date
- Time
- Location
- Expected volunteering time

Actions:

- Manage event
- Attendance

Example:

```text
NEXT EVENT

Humanitarian Bazaar

17 October 2026
10:00 – 16:00
Gradski trg

Expected volunteering time: 6 hours

[ Manage event ]
[ Attendance ]
```

RSVP statistics are not shown on the dashboard.

---

## 13.2 Volunteer Actions

Display:

```text
VOLUNTEERS

[ View volunteers ]
[ Add volunteer ]
```

The dashboard does not show active/inactive volunteer counts.

---

## 13.3 Event Actions

Display:

```text
EVENTS

[ View all events ]
[ Create event ]
```

---

# 14. Admin Volunteers Page

The Volunteers page displays active volunteers by default.

It contains:

- Add volunteer button
- Volunteer search
- Active volunteer list
- Separate access to inactive volunteers

Example:

```text
VOLUNTEERS

[ Add volunteer ]

Search volunteers...

ACTIVE

Amina Hadžić
Prva gimnazija Zenica
81 hours

[ View / Manage ]

INACTIVE

[ Show inactive volunteers ]
```

Inactive volunteers are not mixed into the normal operational volunteer list.

---

# 15. Add Volunteer

The Add Volunteer form contains:

- First name
- Last name
- Year of birth
- School
- Volunteering since year

Example:

```text
ADD VOLUNTEER

First name
[ ]

Last name
[ ]

Year of birth
[ ]

School
[ ]

Volunteering since
[ 2024 ]

[ Create volunteer ]
```

After creation, the system:

1. Creates the volunteer profile
2. Generates the username
3. Generates a temporary password
4. Marks the account as requiring a password change
5. Activates the account

A credential screen is then displayed.

Example:

```text
VOLUNTEER CREATED

Amina Hadžić

Username
ahadzic

Temporary password
K7mP4xQ2

The volunteer will be required to change
this password when they first sign in.

[ Copy credentials ]

[ View volunteer ]
[ Back to volunteers ]
```

---

# 16. Admin Volunteer Details

The Admin Volunteer Details page displays:

- Profile picture
- Name
- Active/inactive status
- Username
- School
- Year of birth
- Volunteering since year
- Total volunteering hours
- Volunteering history
- Account actions

Example:

```text
AMINA HADŽIĆ

[ Profile picture ]

Active volunteer

PERSONAL INFORMATION

Username
ahadzic

School
Prva gimnazija Zenica

Year of birth
2008

Volunteer since
2024

[ Edit information ]

VOLUNTEERING

Total volunteering time
81 hours

[ View volunteering history ]

ACCOUNT

[ Reset password ]

[ Deactivate volunteer ]
```

---

## 16.1 Admin Volunteer Editing

Admins may edit:

- First name
- Last name
- School
- Year of birth
- Volunteering since year

Editing a volunteer's name does not automatically change their username.

Admins must not directly edit a volunteer's total volunteering hours.

If hours are incorrect, the relevant attendance record must be corrected instead.

---

## 16.2 Volunteer Deactivation

Deactivation prevents login and future event participation.

Historical data remains preserved.

Existing historical RSVP and attendance records remain intact.

Inactive volunteers can be reactivated.

---

# 17. Admin Events Page

The Admin Events page uses a chronological list.

It contains:

- Create event button
- Upcoming events
- Past events

There is no search, filtering, year selector, or pagination complexity in V1.

---

## 17.1 Upcoming Events

Upcoming events are ordered nearest first.

Each event provides:

- Event information
- Manage Event button
- Attendance button

---

## 17.2 Past Events

Past events are ordered newest first.

Each event provides:

- View Event button
- Attendance button

Cancelled events remain visible where appropriate.

---

# 18. Create Event

The Create Event form contains:

- Event name
- Start date
- Start time
- End date
- End time
- Location
- Expected volunteering time
- RSVP deadline
- Description

Description is optional and may be empty.

Example:

```text
CREATE EVENT

Event name
[ ]

Start
[ Date ] [ Time ]

End
[ Date ] [ Time ]

Location
[ ]

Expected volunteering time
[ 6.0 ] hours

RSVP deadline
[ Date ] [ Time ]

Description
[ ]

[ Create event ]
```

---

## 18.1 RSVP Deadline Default

When an event start time is selected, the RSVP deadline should default to approximately 24 hours before the event start.

The admin may change this deadline.

---

## 18.2 Expected Volunteering Time

Expected volunteering time may differ from the total event duration.

Example:

Event duration:

10:00–16:00

Expected volunteering time:

5 hours

Decimal values must be supported.

For V1, volunteering time should support practical half-hour increments such as:

- 4
- 4.5
- 5
- 5.5

The underlying implementation should still use a numeric representation that can safely support future changes in precision.

---

## 18.3 Validation

The system should prevent invalid event data.

Examples:

- End timestamp before or equal to start timestamp
- RSVP deadline after the event start
- Negative expected hours
- Missing event name
- Missing location

An RSVP deadline equal to the event start is allowed.

Overnight or multi-day events are valid as long as:

`ends_at > starts_at`

If an admin changes the event start time and the existing RSVP deadline becomes invalid, the admin must correct the RSVP deadline before saving.

---

# 19. Event Timezone

All Medica events use the `Europe/Sarajevo` timezone.

The system must not hardcode CET because Bosnia uses daylight saving time.

Admins do not manually select a timezone.

---

# 20. Manage Event

The Manage Event page displays:

- Event name
- Status
- Date
- Start and end time
- Location
- Expected volunteering time
- RSVP deadline
- Description

Actions include:

- Edit event
- Manage attendance
- Cancel event
- Delete event
- Restore event when applicable

---

# 21. Editing Events

Admins may edit event information after volunteers have already submitted RSVPs.

Existing RSVP responses remain intact.

Changing the event's date, time, location, description, or other general information does not clear existing RSVP responses.

Volunteers should immediately see the updated event information.

If the RSVP deadline is moved:

- Into the future after previously closing, RSVP reopens
- Into the past, RSVP closes immediately

If the new event timing makes the RSVP deadline invalid, saving must be blocked until the deadline is corrected.

---

# 22. Expected Hours Locking

Expected volunteering time can be edited freely until actual volunteer hours have been recorded.

Once any volunteer receives actual attendance hours for the event, the expected volunteering time becomes permanently locked.

Example:

Expected time: 5 hours

Before attendance records exist:

Admin may change it to 6 hours.

After the first volunteer receives actual hours:

Expected time is locked.

Individual actual hours remain editable.

Changing an attendance status later does not unlock the event's expected volunteering time.

---

# 23. Event Lifecycle

The system should not rely on manually marking events as `past`.

The event's UI state should be derived from timestamps.

Possible persisted event states include:

- Scheduled
- Cancelled before start
- Cancelled after start

UI states such as:

- Upcoming
- Happening now
- Past

are calculated automatically.

An event becomes operationally past after its end time plus a two-hour buffer.

Attendance records may still be edited after this point.

---

# 24. RSVP System

Each volunteer can have at most one RSVP record per event.

The database must enforce uniqueness for:

`event + volunteer`

Possible states:

- Yes
- No
- No response

A volunteer may freely change between Yes and No until the RSVP deadline.

After the deadline, the response becomes locked for the volunteer.

Admins may override an RSVP after the deadline if necessary.

RSVP status never determines whether attendance may later be recorded.

A person who answered `No` or did not respond may still attend and receive volunteering hours.

Deactivating a volunteer does not erase their existing RSVP records.

---

# 25. Public RSVP Information

Regular volunteers can only see:

- Total number of volunteers who confirmed attendance

They may not see:

- Names of attending volunteers
- Names of volunteers who declined
- Names of volunteers who did not respond

Admins may see all RSVP identities.

---

# 26. Attendance System

Attendance is separate from RSVP.

Each volunteer can have at most one attendance record per event.

The database must enforce uniqueness for:

`event + volunteer`

Internal attendance states are:

- Not recorded
- Attended
- Absent

`Not recorded` must remain distinct from `Absent`.

---

# 27. Admin Attendance Page

The Attendance page uses a table-oriented layout to allow admins to process volunteers quickly.

The page displays:

- Event information
- Expected volunteering time
- Attendance completion counter
- Add unexpected attendee button
- RSVP-grouped attendance table

Example:

```text
ATTENDANCE

Expected volunteering time: 6 hours

Attendance recorded: 16 / 22

[ Add unexpected attendee ]

COMING

Volunteer        Attendance       Hours      Comment
Amina Hadžić     Attended         6.0        Add
Sara Kovač       Absent           0          Add

NOT COMING

Lejla Imamović   Attended         4.5        Add

NO RESPONSE

Nina Alić        Not recorded     —          Add
```

The three RSVP groups are:

- Coming
- Not coming
- No response

All relevant active volunteers remain eligible for attendance regardless of RSVP status.

---

# 28. Attendance Behavior

## 28.1 Attended

When `Attended` is selected:

- Hours automatically default to the event's expected volunteering time
- Admin may change the actual hours
- Optional admin-only comment can be added

Example:

Expected time: 6 hours

Actual time:

5.5 hours

A volunteer may be marked `Attended` with `0` credited hours if there is a legitimate reason.

Actual hours may never be negative.

---

## 28.2 Absent

When `Absent` is selected:

- Actual hours automatically become `0`
- Hours input is disabled

---

## 28.3 Not Recorded

`Not recorded` means attendance has not yet been processed.

It does not count as either attended or absent.

No volunteering hours are counted.

---

## 28.4 Attendance State Changes

Attendance transitions must behave consistently.

```text
Not recorded → Attended
Default hours = event expected hours

Not recorded → Absent
Hours = 0

Attended → Absent
Hours = 0

Absent → Attended
Default hours = event expected hours

Attended → Not recorded
Clear actual hours

Absent → Not recorded
Clear actual hours
```

If the admin changes from `Absent` back to `Attended`, expected hours are used as the default again, but the admin may edit the value afterward.

---

# 29. Attendance Completion Counter

The Attendance page displays:

```text
Attendance recorded: X / Y
```

Example:

```text
Attendance recorded: 16 / 22
```

No separate `6 not recorded` counter is necessary.

For V1:

- `X` = number of volunteers whose attendance has been explicitly recorded as either `Attended` or `Absent`
- `Y` = number of volunteers currently relevant to the attendance workflow for that event

Inactive volunteers who are not part of that event must not inflate the denominator.

Unexpected attendees explicitly added to the event are included in the attendance workflow.

---

# 30. Unexpected Attendees

The system must support unexpected attendees.

An admin can select:

`Add unexpected attendee`

The admin may search/select an active volunteer who did not originally confirm attendance.

The volunteer may then be marked as attended and receive actual hours.

Their original RSVP status remains historically accurate.

If the selected volunteer already exists in the attendance table under `Not coming` or `No response`, the system must not create a duplicate record.

Instead, the UI should navigate to or highlight that existing row.

---

# 31. Attendance Comments

Admins may add optional comments to attendance records.

Examples:

- Left 30 minutes early
- Stayed an extra 1.5 hours
- Arrived late

Attendance comments are visible only to admins.

Volunteers never see these comments.

---

# 32. Attendance Saving

Attendance changes should save immediately or automatically.

The system should not rely on one large `Save Attendance` button.

This prevents the admin from losing progress if the page is closed during an event.

A small saved-state indicator may be used.

---

# 33. Attendance Corrections

Attendance remains editable after the event.

Admins may later correct:

- Attendance status
- Actual hours
- Admin comment

There is no finalization lock.

Historical corrections must remain possible.

If multiple admins edit the same attendance record at nearly the same time, V1 may use a simple latest-saved-value-wins approach.

Advanced conflict resolution is not required.

---

# 34. Volunteering Hours

Volunteering hours are calculated from actual attendance records.

The system must not maintain a manually editable authoritative `total_hours` value.

A volunteer's total volunteering time is derived from the sum of their actual attendance hours.

Conceptually:

```text
SUM(attendance.hours)
```

Only attended records contribute positive volunteering hours.

This ensures totals cannot drift away from the underlying history.

---

# 35. Yearly Hours

V1 does not display or require yearly volunteering-hour totals.

Only total all-time volunteering time is shown.

Because attendance records retain event dates, yearly totals can be calculated later without redesigning the database.

---

# 36. Event Cancellation

Events can be cancelled by admins.

A cancelled event must be clearly marked as cancelled.

While an event is cancelled:

- Volunteer RSVP editing is disabled
- Attendance remains accessible to admins
- Historical RSVP data remains preserved
- The event may be restored
- Permanent deletion remains subject to attendance-record rules

There are conceptually two situations.

## 36.1 Cancelled Before Starting

The event did not take place.

No attendance or hours exist.

RSVP data is not historically important.

The event may be deleted permanently.

## 36.2 Cancelled After Starting

The event began or partially occurred.

Attendance and volunteering hours may exist.

Those records must remain preserved.

The event cannot be deleted if attendance/hour records exist.

---

# 37. Restore Event

Cancelled events can be restored.

This supports accidental cancellations or events that are reinstated.

A cancelled event may be restored at any time.

Restoring an event:

- Preserves all existing RSVP records
- Preserves attendance records
- Preserves event information
- Does not automatically reopen RSVP

If the RSVP deadline has already passed, RSVP remains closed unless the admin also changes the deadline.

The event's timestamps still determine whether it is displayed as upcoming, happening now, or past.

---

# 38. Event Deletion

An event may be permanently deleted only when there are no attendance/hour records.

RSVP records alone do not prevent deletion.

If an event is deleted:

- Associated RSVP records should be deleted automatically
- No orphaned RSVP records may remain

If attendance records exist, deletion must be blocked.

Example message:

```text
This event cannot be deleted because
attendance has already been recorded.
```

Deletion must require a confirmation action because it is permanent.

---

# 39. Profile Pictures

Volunteers may upload or change their own profile picture.

Admins can view profile pictures.

Profile pictures must not become part of a publicly browsable volunteer directory.

If a profile picture does not exist, a default avatar or initials should be displayed.

---

# 40. Privacy Principles

Many volunteers may be minors.

The application should therefore store only information Medica actually needs.

V1 stores:

- Name
- Username
- Birth year
- School
- Volunteering since year
- Profile picture
- Event participation
- Volunteering hours

The application should not collect unnecessary information such as:

- Full birth date
- Home address
- Phone number
- Other unrelated personal information

unless a future legitimate requirement exists.

---

# 41. Access Control

Permissions must be enforced on the server/database side, not only through hidden UI elements.

Volunteers must not be able to query:

- Other volunteer profiles
- Other volunteer attendance histories
- RSVP identities of other volunteers
- Admin-only comments

Volunteers may access only their own private volunteer information.

Admins may access the information required to manage the organization.

---

# 42. Security Principles

The application must follow basic security requirements.

- Never store plaintext passwords
- Never expose Supabase service-role credentials in the browser
- Validate server inputs
- Use database-level authorization where appropriate
- Use Supabase Row Level Security
- Restrict volunteer data access appropriately
- Protect admin-only operations server-side
- Enforce unique usernames where required
- Enforce one RSVP record per volunteer per event
- Enforce one attendance record per volunteer per event

---

# 43. Data Integrity Principles

The application should prefer preserving historical accuracy over silently rewriting history.

Examples:

- Changing a volunteer's name must not rewrite their username automatically
- Deactivating a volunteer must not erase historical attendance
- Changing event information must not erase RSVP responses
- Adding an unexpected attendee must not rewrite their original RSVP
- Restoring an event must not reset its RSVP records
- Volunteering totals must come from attendance records rather than a separately edited total

Related records must not be left orphaned when parent data is deleted.

---

# 44. Out of Scope for V1

The following features are intentionally excluded from V1:

- Leaderboard
- Push notifications
- Email notifications
- Messaging
- Chat
- Social feed
- Comments between volunteers
- Likes/reactions
- Google Calendar integration
- Native mobile applications
- Public volunteer registration
- Complex admin permission levels
- Event search
- Event filtering
- Event calendar view
- Yearly volunteering-hour totals
- Detailed analytics dashboards
- Badges or achievements
- Advanced concurrent-edit conflict handling
- Permanent volunteer-account deletion

These may be considered in future versions if a genuine need appears.

---

# 45. Future Features

Possible future additions include:

- Current-year volunteer leaderboard
- Add to Google Calendar
- CSV or Excel exports
- Volunteer certificates
- Yearly statistics
- Event search/filtering
- Calendar-based event view
- More advanced reporting
- Volunteer achievement systems

Future features must not influence V1 complexity unless required by the current product.

---

# 46. V1 Screen Map

## Volunteer

```text
Volunteer
├── Home
├── Events
│   └── Event Details
└── Profile
    └── Volunteering History
```

## Admin

```text
Admin
├── Dashboard
├── Volunteers
│   ├── Add Volunteer
│   └── Volunteer Details
├── Events
│   ├── Create Event
│   ├── Manage Event
│   └── Attendance
└── Admins
    ├── Add Admin
    └── Admin Details
```

---

# 47. Core Product Principle

V1 should remain small, reliable, and easy to understand.

New features should not be added simply because they are technically possible.

The priority is:

1. Reliable volunteer account management
2. Clear event organization
3. Simple RSVP handling
4. Fast attendance recording
5. Accurate volunteering-hour history

Everything else is secondary.

When a behavior is not explicitly defined in this specification, implementation should favor the simplest behavior that:

- Preserves historical data
- Does not expose private information
- Does not create duplicate records
- Does not silently alter volunteering hours
- Does not introduce a new product feature

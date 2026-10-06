# Medica Web App — V1 Technical Architecture

## 1. Purpose

This document defines the technical architecture for V1 of the Medica Web App.

It translates the behavior defined in `docs/PRODUCT_SPEC.md` into concrete technical decisions for authentication, database structure, permissions, storage, routing, server-side operations, maintenance, and deployment.

This document should be treated as the technical source of truth for V1.

If a product behavior is unclear, `docs/PRODUCT_SPEC.md` takes precedence.
If a technical implementation detail is unclear, this document takes precedence.

---

# 2. Architecture Goals

V1 should prioritize:

1. Simplicity
2. Low operating cost
3. Privacy
4. Data integrity
5. Easy maintenance
6. Clear separation between volunteer and admin capabilities
7. Avoiding unnecessary infrastructure

The application is intended for approximately 30–40 active volunteers and a small number of admins.

The architecture should not be designed for large-scale enterprise usage.

---

# 3. Technology Stack

## 3.1 Application

- Next.js
- React
- TypeScript
- Tailwind CSS

The application will be deployed on the existing Hetzner VPS.

The expected production hostname is:

`medica.kerim.ba`

Nginx may be used as the reverse proxy in front of the Next.js application.

---

## 3.2 Backend Services

Supabase Free will provide:

- PostgreSQL database
- Authentication
- Row Level Security
- Private file storage for profile pictures

No separate Express backend, MongoDB, Firebase, Redis, or other backend service is required for V1.

---

## 3.3 Cost Principle

V1 should use the existing VPS and Supabase Free tier only.

No additional paid infrastructure should be introduced unless a future requirement makes it necessary.

---

# 4. High-Level Deployment Model

```text
User browser
     |
     v
medica.kerim.ba
     |
     v
Nginx
     |
     v
Next.js application on existing VPS
     |
     +-----------------------------+
     |                             |
     v                             v
Supabase Auth                Supabase PostgreSQL
                                   |
                                   v
                            Supabase Storage
```

The browser may communicate directly with Supabase for safe volunteer operations protected by RLS.

Privileged operations must go through the Next.js server.

---

# 5. Authentication Model

## 5.1 Shared Login

Admins and volunteers use one shared login page.

The visible login form contains:

- Username
- Password

The user never needs to enter an email address.

After authentication:

- Volunteer → volunteer application
- Admin → admin application
- `must_change_password = true` → forced password-change page first
- inactive account → access denied

---

## 5.2 Internal Supabase Identity

Supabase Auth will use an internal email-style identity derived from the username.

Example:

```text
Visible username:
ahadzic

Internal Supabase identity:
ahadzic@medica.internal
```

The internal identity is implementation-only and must not be shown in normal UI.

No real email delivery depends on the internal identity.

---

## 5.3 Canonical User ID

The Supabase Auth user UUID is the canonical account identifier across the entire application.

```text
auth.users.id
      |
      +--> accounts.id
      +--> volunteer_profiles.user_id
      +--> event_rsvps.volunteer_id
      +--> attendance.volunteer_id
```

No separate numeric volunteer or admin ID system should be created.

---

# 6. Account Model

There are two account roles:

- `volunteer`
- `admin`

Both authenticate through Supabase Auth.

Shared account information belongs in `accounts`.

Volunteer-only data belongs in `volunteer_profiles`.

No separate `admin_profiles` table is required in V1.

---

# 7. Database Tables

The V1 database contains five core application tables:

1. `accounts`
2. `volunteer_profiles`
3. `events`
4. `event_rsvps`
5. `attendance`

No audit-history tables are required in V1.

---

# 8. Table: accounts

Purpose:

Stores shared application-level identity, role, and account state.

```text
accounts
├── id                    uuid PRIMARY KEY
├── username              text UNIQUE NOT NULL
├── first_name            text NOT NULL
├── last_name             text NOT NULL
├── role                  text NOT NULL
├── is_active             boolean NOT NULL DEFAULT true
├── must_change_password  boolean NOT NULL DEFAULT true
├── created_at            timestamptz NOT NULL DEFAULT now()
└── updated_at            timestamptz NOT NULL DEFAULT now()
```

## 8.1 Relationships

`accounts.id` references:

`auth.users.id`

The account UUID must be identical to the corresponding Supabase Auth user UUID.

---

## 8.2 Role Constraint

`role` is stored as text with a database CHECK constraint.

Allowed values:

```text
admin
volunteer
```

Conceptually:

```sql
CHECK (role IN ('admin', 'volunteer'))
```

Custom PostgreSQL enums are not required in V1.

---

## 8.3 Username Rules

Usernames:

- are unique
- are stored in lowercase canonical form
- are used to derive the internal Supabase identity
- are never used as a password
- are visible to the account owner and admins

Volunteer usernames are generated automatically.

Admin usernames are chosen manually.

Changing a volunteer's name does not automatically change the username.

Admin usernames may be changed by an authorized admin operation.

If an admin username changes, the application must update both:

- `accounts.username`
- the corresponding internal Supabase Auth identity

The change must be handled as one protected server-side workflow.

---

# 9. Table: volunteer_profiles

Purpose:

Stores data that only applies to volunteer accounts.

```text
volunteer_profiles
├── user_id                  uuid PRIMARY KEY
├── school                   text NULL
├── birth_year               integer NOT NULL
├── volunteering_since_year  integer NOT NULL
├── avatar_path              text NULL
├── created_at               timestamptz NOT NULL DEFAULT now()
└── updated_at               timestamptz NOT NULL DEFAULT now()
```

`user_id` references `accounts.id`.

Only accounts with `role = 'volunteer'` should have a row in this table.

---

## 9.1 Profile Validation

The system should enforce:

- `birth_year <= current year`
- `volunteering_since_year <= current year`
- `volunteering_since_year >= birth_year`

The UI may perform additional plausibility validation.

The database should avoid overly restrictive age assumptions.

`school` is optional.

---

# 10. Table: events

Purpose:

Stores Medica events.

```text
events
├── id                 uuid PRIMARY KEY
├── title              text NOT NULL
├── description        text NULL
├── location           text NOT NULL
├── starts_at          timestamptz NOT NULL
├── ends_at            timestamptz NOT NULL
├── rsvp_deadline      timestamptz NOT NULL
├── expected_hours     numeric(5,2) NOT NULL
├── status             text NOT NULL DEFAULT 'scheduled'
├── created_by         uuid NOT NULL
├── created_at         timestamptz NOT NULL DEFAULT now()
└── updated_at         timestamptz NOT NULL DEFAULT now()
```

---

## 10.1 Event Status

Allowed stored statuses:

```text
scheduled
cancelled_before_start
cancelled_after_start
```

Conceptually:

```sql
CHECK (
  status IN (
    'scheduled',
    'cancelled_before_start',
    'cancelled_after_start'
  )
)
```

The admin does not manually choose which cancellation status applies.

When an event is cancelled:

```text
current time < starts_at
→ cancelled_before_start

current time >= starts_at
→ cancelled_after_start
```

Restoring an event sets:

```text
status = scheduled
```

---

## 10.2 Derived Event States

The database does not store:

- upcoming
- happening now
- past

These are derived from timestamps.

Conceptually:

```text
starts_at > now
→ Upcoming

starts_at <= now <= ends_at + 2 hours
→ Happening / active operational window

now > ends_at + 2 hours
→ Past
```

Cancellation status overrides the normal scheduled presentation where appropriate.

---

## 10.3 Event Timezone

Event input is interpreted using:

`Europe/Sarajevo`

The application must not hardcode CET or a fixed UTC offset.

The stored timestamp values should use timezone-aware timestamps.

---

## 10.4 Event Constraints

The database/application must enforce:

```text
ends_at > starts_at
rsvp_deadline <= starts_at
expected_hours >= 0
```

Overnight and multi-day events are valid.

If event start time is changed and the RSVP deadline becomes invalid, saving must be blocked until the RSVP deadline is corrected.

---

## 10.5 Expected Hours Lock

`expected_hours` may be edited until the first volunteer is recorded as `attended`.

An `absent` attendance row does not lock `expected_hours`.

Once any volunteer has an attendance row with `status = 'attended'`, `expected_hours` becomes permanently locked, including when that attended row has `actual_hours = 0`.

Later attendance corrections do not unlock `expected_hours`.

This rule must be enforced server-side for admin event updates.

---

# 11. Table: event_rsvps

Purpose:

Stores explicit volunteer RSVP responses.

```text
event_rsvps
├── id            uuid PRIMARY KEY
├── event_id      uuid NOT NULL
├── volunteer_id  uuid NOT NULL
├── response      text NOT NULL
├── created_at    timestamptz NOT NULL DEFAULT now()
└── updated_at    timestamptz NOT NULL DEFAULT now()
```

Allowed `response` values:

```text
yes
no
```

Conceptually:

```sql
CHECK (response IN ('yes', 'no'))
```

---

## 11.1 No Response

There is no stored `no_response` row.

Instead:

```text
no RSVP row
→ No response
```

This avoids creating meaningless RSVP records for every volunteer/event combination.

---

## 11.2 RSVP Uniqueness

Each volunteer may have at most one RSVP row per event.

```sql
UNIQUE (event_id, volunteer_id)
```

---

## 11.3 RSVP Semantics

The three logical RSVP states are:

```text
Yes
No
No response
```

`No response` is distinct from explicit `No`.

For attendance workflow purposes:

- `Yes` = expected
- `No` = explicitly not coming
- `No response` = no RSVP submitted

A volunteer with `No` or `No response` may still attend and receive hours.

---

# 12. Table: attendance

Purpose:

Stores actual attendance outcomes and credited volunteering hours.

```text
attendance
├── id             uuid PRIMARY KEY
├── event_id       uuid NOT NULL
├── volunteer_id   uuid NOT NULL
├── status         text NOT NULL
├── actual_hours   numeric(5,2) NOT NULL
├── comment        text NULL
├── marked_by      uuid NOT NULL
├── created_at     timestamptz NOT NULL DEFAULT now()
└── updated_at     timestamptz NOT NULL DEFAULT now()
```

Allowed `status` values:

```text
attended
absent
```

Conceptually:

```sql
CHECK (status IN ('attended', 'absent'))
```

---

## 12.1 Not Recorded

There is no stored `not_recorded` row.

Instead:

```text
no attendance row
→ Not recorded
```

This avoids pre-generating attendance records for every volunteer.

---

## 12.2 Attendance Uniqueness

Each volunteer may have at most one attendance row per event.

```sql
UNIQUE (event_id, volunteer_id)
```

---

## 12.3 Attendance Constraints

Rules:

```text
actual_hours >= 0

status = absent
→ actual_hours = 0

status = attended
→ actual_hours >= 0
```

An attended volunteer may legitimately receive `0` credited hours.

Negative hours are never allowed.

---

## 12.4 Attendance State Changes

UI/database behavior:

```text
Not recorded → Attended
Create attendance row
Default actual_hours = event expected_hours

Not recorded → Absent
Create attendance row
actual_hours = 0

Attended → Absent
Update row
actual_hours = 0

Absent → Attended
Update row
Default actual_hours = event expected_hours

Attended → Not recorded
Delete attendance row

Absent → Not recorded
Delete attendance row
```

No attendance row is the canonical representation of `Not recorded`.

---

## 12.5 marked_by

`marked_by` references the admin account UUID that most recently changed the attendance row.

A full audit log is intentionally out of scope for V1.

If multiple admins edit the same row at nearly the same time, V1 may use last-saved-value-wins behavior.

---

# 13. Volunteering Hours

There is no `total_hours` field.

A volunteer's total volunteering time is always derived from attendance rows.

Conceptually:

```sql
SUM(attendance.actual_hours)
WHERE attendance.status = 'attended'
```

This is the single source of truth.

Changing an attendance record automatically changes the calculated total.

Changing an event's expected hours does not retroactively rewrite recorded actual hours.

V1 does not require yearly hour totals.

---

# 14. Foreign Keys and Delete Behavior

The architecture should preserve historical data whenever possible.

## 14.1 accounts

```text
accounts.id
→ auth.users.id
→ deletion not used in normal V1 workflows
```

Application accounts are deactivated instead of deleted.

---

## 14.2 volunteer_profiles

```text
volunteer_profiles.user_id
→ accounts.id
→ ON DELETE RESTRICT
```

---

## 14.3 events.created_by

```text
events.created_by
→ accounts.id
→ ON DELETE RESTRICT
```

Deactivating the admin does not affect historical event ownership.

---

## 14.4 event_rsvps.event_id

```text
event_rsvps.event_id
→ events.id
→ ON DELETE CASCADE
```

If an event is permanently deleted, its RSVP rows are automatically removed.

---

## 14.5 event_rsvps.volunteer_id

```text
event_rsvps.volunteer_id
→ accounts.id
→ ON DELETE RESTRICT
```

Historical RSVP data survives account deactivation.

---

## 14.6 attendance.event_id

```text
attendance.event_id
→ events.id
→ ON DELETE RESTRICT
```

This supports the rule that an event with attendance records cannot be permanently deleted.

---

## 14.7 attendance.volunteer_id

```text
attendance.volunteer_id
→ accounts.id
→ ON DELETE RESTRICT
```

Historical attendance and volunteering hours survive volunteer deactivation.

---

## 14.8 attendance.marked_by

```text
attendance.marked_by
→ accounts.id
→ ON DELETE RESTRICT
```

Deactivating an admin does not remove their historical reference.

---

# 15. Event Deletion Rules

An event may be permanently deleted only if it has no attendance rows.

RSVP rows alone do not prevent deletion.

Therefore:

```text
Event has RSVP rows only
→ permanent deletion allowed
→ RSVP rows cascade-delete

Event has attendance rows
→ permanent deletion blocked
```

Deletion requires explicit admin confirmation.

---

# 16. Account Deactivation

Account deactivation is reversible.

It is not deletion.

For both volunteers and admins:

```text
accounts.is_active = false
+
Supabase Auth access disabled/banned
```

Result:

- login is blocked
- existing sessions must not grant normal app access
- historical records remain
- account can be reactivated later

Reactivation reverses both the application-level and Auth-level access block.

---

## 16.1 Volunteer Deactivation

Deactivation does not modify:

- RSVP history
- attendance history
- volunteering hours
- profile history
- event references

Inactive volunteers remain visible to admins where historical context requires them.

---

## 16.2 Admin Deactivation

Deactivating an admin does not modify:

- events they created
- attendance rows they marked
- historical references

The last active admin may never be deactivated.

An admin may not deactivate their own currently logged-in account.

---

## 16.3 First Admin Bootstrap

The first admin account is created manually during initial system setup.

This is a one-time bootstrap step.

After the first admin exists:

- all future admin accounts are created through the application
- the normal admin-creation permissions apply
- no public or self-service admin registration exists

The initial manual setup should create:

- the Supabase Auth user
- the matching `accounts` row
- `role = 'admin'`
- `is_active = true`

The first admin may initially use a temporary password and should be required to change it after first login.

---

# 17. Temporary Password Model

Supabase Auth stores and verifies all real passwords.

The application database never stores passwords.

`accounts.must_change_password` controls the first-login/reset flow.

---

## 17.1 New Account Flow

```text
Admin creates user
→ Next.js server verifies active admin
→ server creates Supabase Auth user
→ internal identity = username@medica.internal
→ temporary password is set
→ accounts row created
→ volunteer_profiles row created if applicable
→ must_change_password = true
→ admin receives temporary credentials once
```

Because Supabase Auth creation and application-table creation are not one database transaction, the server must clean up partial account creation.

If the Auth user is created successfully but the required `accounts` or `volunteer_profiles` creation fails:

- the newly created Auth user should be deleted/rolled back by the server
- the operation should return an error
- no partially created account should remain

---

## 17.2 First Login

```text
User signs in
→ authentication succeeds
→ app reads account
→ must_change_password = true
→ redirect to /change-password
→ user chooses new password
→ Supabase Auth password updated
→ must_change_password = false
→ route to normal application
```

---

## 17.3 Password Reset

```text
Admin initiates reset
→ server verifies active admin
→ server generates/sets new temporary password
→ previous password becomes invalid
→ must_change_password = true
→ temporary credentials shown to admin
```

There is no self-service email password reset in V1.

---

# 18. Authorization Strategy

Authorization must not depend only on hidden UI elements.

Security is enforced through:

1. Supabase Row Level Security
2. Next.js server-side admin checks
3. application route guards
4. account `is_active` checks

---

## 18.1 RLS Helper Functions

RLS policies should use small reusable database helper functions rather than embedding the same account-role logic repeatedly.

Suggested helpers:

```text
is_active_user()
is_admin()
```

Conceptually:

```text
is_active_user()
→ auth.uid() has an `accounts` row
→ accounts.is_active = true

is_admin()
→ is_active_user() = true
→ accounts.role = 'admin'
```

These helpers should derive authorization from `auth.uid()` and the `accounts` table.

V1 should not depend on custom JWT role claims for authorization.

This keeps role changes and account deactivation immediately tied to the database source of truth.

---

# 19. RLS Permission Model

## 19.1 accounts

Volunteer:

- read own account row
- cannot read other accounts
- cannot directly change role
- cannot directly change username
- cannot directly change `is_active`
- cannot directly change `must_change_password`

Admin:

- may read accounts required for management
- privileged modifications occur through protected server-side operations

---

## 19.2 volunteer_profiles

Volunteer:

- read own profile
- update own allowed fields only
- cannot read another volunteer's profile

Allowed volunteer-editable data:

- school
- own avatar-related value through controlled upload flow

Admin:

- read all volunteer profiles
- update protected volunteer information through admin flows

---

## 19.3 events

Active volunteer:

- read event information required by the volunteer application

Volunteer cannot:

- create events
- edit events
- cancel events
- restore events
- delete events

Admin:

- manages events through protected server-side operations

---

## 19.4 event_rsvps

Volunteer:

- read own RSVP only
- create/update own RSVP only
- cannot read other volunteers' RSVP rows

Volunteer RSVP writes must also satisfy:

- account active
- event scheduled
- RSVP deadline not passed

Admin:

- can read all RSVP identities
- can override RSVP when needed
- privileged overrides occur through server-side admin operations

---

## 19.5 attendance

Volunteer:

- cannot insert attendance
- cannot update attendance
- cannot delete attendance
- may access only safe attendance/history data for themselves
- must never receive admin-only comment data

Admin:

- manages attendance through protected server-side operations

---

# 20. Safe Volunteer Read Models

Some volunteer-facing data must be exposed without revealing private raw rows.

Use safe database views/functions or equivalent protected read models where they simplify privacy.

---

## 20.1 Confirmed Attendee Count

Volunteers may see:

```text
confirmed_count
```

They may not see:

- attendee identities
- declined volunteer identities
- no-response volunteer identities

A safe aggregate view/function may return:

```text
event_id
confirmed_count
```

The raw RSVP table is not exposed broadly to volunteers.

---

## 20.2 Volunteer Attendance History

Volunteer history should expose only fields required by the UI, such as:

```text
event_id
event_title
event_date
attendance_status
actual_hours
```

It must not expose:

```text
comment
marked_by
```

Admin-only comments remain private.

---

# 21. Attendance Workflow Set

The attendance page distinguishes RSVP categories:

- Coming
- Not coming
- No response

For the attendance completion counter:

```text
X = volunteers in the attendance workflow
    whose attendance has been recorded
    as Attended or Absent

Y = Coming
  + Not coming
  + Unexpected attendees
```

`No response` volunteers do not count toward `Y` by default.

If a `No response` volunteer actually appears, the admin adds them as an unexpected attendee and they then count toward the workflow total.

---

## 21.1 Unexpected Attendees

An unexpected attendee:

- must be an active volunteer
- may originally have RSVP'd `No`
- may originally have had `No response`
- retains the original RSVP state
- receives an attendance row if attendance is recorded
- must not create duplicate RSVP or attendance records

If already represented in the attendance workflow, the UI should navigate to/highlight the existing entry rather than creating a duplicate.

---

# 22. Direct Supabase vs Next.js Server

The application uses a hybrid model.

---

## 22.1 Direct Supabase Operations

Safe volunteer operations protected by RLS may communicate directly with Supabase.

Examples:

- read own account/profile
- update own school
- read events
- read own RSVP
- create/update own RSVP
- read safe own attendance/history
- upload/replace own profile picture through controlled storage policies

---

## 22.2 Server-Side Privileged Operations

Sensitive operations must go through Next.js server actions or API routes.

Examples:

- create volunteer account
- create admin account
- reset password
- deactivate/reactivate account
- change admin username/internal auth identity
- create event
- edit event
- cancel event
- restore event
- delete event
- override RSVP
- create/update/delete attendance
- edit hours/comments
- manage admin accounts

Pattern:

```text
Browser
   |
   v
Next.js server action / API route
   |
   v
Verify authenticated active admin
   |
   v
Privileged Supabase operation
```

The Supabase service-role credential must never be exposed to the browser.

---

# 23. Profile Picture Storage

Profile pictures are stored in a private Supabase Storage bucket.

Suggested bucket:

```text
profile-pictures
```

Only the path is stored in the database:

```text
volunteer_profiles.avatar_path
```

---

## 23.1 Access Rules

Volunteer:

- view own profile picture
- upload/change own profile picture
- cannot view other volunteers' profile pictures

Admin:

- view volunteer profile pictures required for management

The bucket is not public.

---

## 23.2 Replacement Behavior

Each volunteer has only one current profile picture.

When replaced:

1. new image is validated
2. new image is resized/compressed
3. new image is uploaded
4. database path is updated
5. previous unused file is deleted

No image history/versioning is kept.

---

## 23.3 Image Validation

Accepted input formats:

- JPG/JPEG
- PNG
- WEBP

Maximum original selected file size:

`5 MB`

Before upload, the client should resize/compress the image to approximately avatar-sized dimensions, such as roughly:

`512 × 512 px`

The implementation may preserve aspect ratio and crop appropriately for avatar presentation.

A compressed JPEG or WEBP representation is preferred.

If no profile image exists, the UI shows initials or a default avatar.

---

# 24. Next.js Route Structure

Suggested route structure:

```text
/
├── login
├── change-password
│
├── home
├── events
│   └── [eventId]
├── profile
│   └── history
│
└── admin
    ├── dashboard
    ├── volunteers
    │   ├── new
    │   └── [volunteerId]
    ├── events
    │   ├── new
    │   └── [eventId]
    │       ├── edit
    │       └── attendance
    └── admins
        ├── new
        └── [adminId]
```

The exact Next.js folder implementation may use route groups/layouts as appropriate, but the user-visible routing model should remain simple.

---

# 25. Route Guards

The application must enforce:

```text
Unauthenticated
→ /login

must_change_password = true
→ /change-password

inactive account
→ deny normal application access

volunteer entering /admin/*
→ deny/redirect

admin entering normal volunteer workflow by mistake
→ route to /admin/dashboard where appropriate
```

Route guards are a convenience/security layer and do not replace database/server authorization.

---

# 26. Updated Timestamps

Tables containing `updated_at` should automatically refresh it when rows change.

Prefer a small reusable PostgreSQL trigger rather than relying on every frontend/server operation to remember to update the timestamp manually.

Tables requiring automatic `updated_at` behavior:

- accounts
- volunteer_profiles
- events
- event_rsvps
- attendance

---

# 27. Error Handling

Server operations should return a consistent structured result.

Conceptually:

```text
success
message
optional field errors
```

UI messages should be short and understandable.

Examples:

```text
Volunteer created successfully.
Could not save attendance.
This username is already in use.
RSVP deadline has passed.
```

Raw PostgreSQL, Supabase, stack trace, or internal server errors must never be displayed directly to end users.

Validation errors should appear next to the relevant field where possible.

Unexpected internal errors should:

- return a generic user-facing message
- be logged server-side for debugging

Destructive actions require confirmation.

---

# 28. Environment Variables and Secrets

Suggested variables:

```text
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY

SUPABASE_SERVICE_ROLE_KEY
SUPABASE_DB_URL

INTERNAL_AUTH_DOMAIN=medica.internal
```

If the current Supabase SDK/project uses a newer public-client key naming convention, the implementation may use the current equivalent while preserving the same public-vs-private separation.

---

## 28.1 Public Values

Only variables explicitly intended for browser use may use `NEXT_PUBLIC_*`.

Public browser configuration must never include privileged credentials.

---

## 28.2 Server-Only Secrets

These remain server-only:

```text
SUPABASE_SERVICE_ROLE_KEY
SUPABASE_DB_URL
```

They must never be sent to the browser.

---

## 28.3 Git

Real environment files must not be committed.

The repository should contain:

```text
.env.example
```

with variable names and placeholders only.

Production values live on the VPS.

---

# 29. Maintenance Schedule

The VPS will run lightweight scheduled maintenance in the `Europe/Sarajevo` timezone.

Daily schedule:

```text
04:00 → Supabase health / keep-alive check
04:20 → PostgreSQL backup
```

The schedule should follow `Europe/Sarajevo` rather than a fixed UTC offset.

---

# 30. Health / Keep-Alive Check

A lightweight health endpoint or scheduled script should perform a harmless database read.

Example concept:

```text
VPS cron
→ Medica health endpoint / script
→ harmless Supabase SELECT
→ success/failure
```

The health check must not:

- create fake events
- create dummy volunteer records
- modify real application data

It serves two purposes:

1. verify the application can reach Supabase
2. generate legitimate periodic database activity

The endpoint should not expose sensitive system information publicly.

---

# 31. Database Backups

The VPS should periodically create PostgreSQL dumps of the Supabase database.

Suggested schedule:

```text
04:20 Europe/Sarajevo
```

Use:

- `pg_dump`
- compression
- local VPS backup directory
- rotating retention

Example:

```text
/backups/medica/
├── medica-2026-10-06.sql.gz
├── medica-2026-10-05.sql.gz
├── medica-2026-10-04.sql.gz
└── ...
```

Do not keep unlimited backup history.

Backup scripts and database credentials must not be committed to the public repository.

V1 database backups cover PostgreSQL data only.

Supabase Storage profile pictures are intentionally not included in the backup strategy because they are non-critical and can be re-uploaded if lost.

---

# 32. Security Principles

V1 should be secure by default without unnecessary enterprise complexity.

Requirements:

- never store plaintext passwords
- never expose service-role credentials to the browser
- use Supabase Auth for password verification
- use RLS for database privacy
- use server-side admin authorization
- restrict private profile pictures
- validate all server-side inputs
- enforce database uniqueness and CHECK constraints
- preserve historical records
- prevent duplicate RSVP records
- prevent duplicate attendance records
- do not rely solely on frontend restrictions

Advanced threat modeling and enterprise security infrastructure are out of scope for V1.

---

# 33. Data Integrity Principles

The database should preserve historical truth.

Examples:

- name edits do not rewrite volunteer usernames
- deactivation does not erase history
- event edits do not erase RSVP responses
- unexpected attendance does not rewrite RSVP history
- attendance corrections change totals automatically
- total volunteering hours are never manually maintained
- event deletion cannot remove attendance history
- admin deactivation does not erase event/attendance references

---

# 34. Concurrency

V1 does not require sophisticated concurrent-edit conflict resolution.

If two admins update the same ordinary record nearly simultaneously:

```text
latest successfully saved value wins
```

This is acceptable for the expected small admin team.

A full audit/history system may be added in a future version if needed.

---

# 35. Out of Scope for V1 Architecture

The following are intentionally not part of V1 architecture:

- separate microservices
- Redis
- message queues
- background worker infrastructure beyond simple scheduled VPS jobs
- external notification infrastructure
- email delivery system
- audit log tables
- advanced role/permission hierarchy
- permanent volunteer deletion
- event analytics warehouse
- dedicated search infrastructure
- separate admin database
- native mobile backend
- paid backup service
- additional paid hosting

---

# 36. Initial Database Relationship Summary

```text
auth.users
    |
    v
accounts
    |
    +--------------------+
    |                    |
    v                    v
volunteer_profiles     events
                         |
                         +--------------------+
                         |                    |
                         v                    v
                    event_rsvps          attendance
```

Additional references:

```text
events.created_by
→ accounts.id

event_rsvps.volunteer_id
→ accounts.id

attendance.volunteer_id
→ accounts.id

attendance.marked_by
→ accounts.id
```

---

# 37. Core V1 Architecture Rule

Prefer the simplest architecture that:

- satisfies `PRODUCT_SPEC.md`
- preserves historical data
- prevents unauthorized access
- keeps privileged credentials server-side
- remains understandable to maintain
- remains within the existing free infrastructure
- does not introduce features that V1 does not need

When an implementation choice is not explicitly defined, choose the simplest option that follows these principles rather than inventing new product behavior.

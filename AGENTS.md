# Medica Web App — Codex Instructions

## Project
This repository contains the Medica volunteer management web application.

Medica is a volunteer organization. The app is a private internal portal for admins and volunteers.

## Source of truth
Before implementing features, read:

- docs/PRODUCT_SPEC.md
- docs/SCREEN_MAP.md
- docs/ARCHITECTURE.md

These documents define the intended product behavior, screens, business rules, database structure, and security model.

Do not invent features that are not described there.

If implementation details are unclear, prefer the simplest solution consistent with the documentation.

## Stack
Use:

- Next.js
- React
- TypeScript
- App Router
- Tailwind CSS
- Supabase for PostgreSQL, Auth, and Storage

Do not introduce additional backend frameworks or databases unless explicitly requested.

## Design
Use the Figma designs as the visual reference.

Primary Medica brand color:
- #16345E

Design principles:
- clean
- minimal
- professional
- responsive
- white/light-gray surfaces
- subtle borders
- limited shadows
- reusable components
- consistent spacing

Do not redesign screens or add UI features unless explicitly requested.

## Architecture
Prefer reusable components.

Suggested structure:

src/
  app/
  components/
    ui/
    layout/
    volunteer/
    admin/
  lib/
  types/

Use server-side code for privileged admin operations.

Never expose:
- Supabase service-role keys
- database credentials
- private server environment variables

## Supabase
Follow the database and RLS design described in docs/ARCHITECTURE.md.

Volunteer-facing operations must respect RLS.

Privileged admin operations should use protected server actions or route handlers.

Do not use custom JWT role claims for authorization.

Roles and active status are determined from the accounts table.

## Timezone
Application business logic should use:

Europe/Sarajevo

## Code quality
Use TypeScript strictly.

Avoid unnecessary abstractions.

Prefer clear, maintainable code over clever code.

After meaningful changes:
- run lint
- run type checking
- fix errors before finishing

Do not silently ignore failing checks.

## Workflow
Work in small milestones.

Do not build the entire application in one change.

Before starting a major feature:
1. inspect the relevant product documentation
2. inspect existing components
3. reuse existing patterns where possible

When a task is complete, summarize:
- what was changed
- important implementation decisions
- anything still unfinished

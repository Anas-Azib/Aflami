# Aflami (أفلامي)

One platform for a short film's journey: create the project, build the team, distribute the work, and publish the film page with full credits.

Graduation project, Faculty of Computing and Information Technology, King Abdulaziz University.
CPIT-498 (requirements and design) → CPIT-499 (implementation and testing).

## Team

| Member | Responsible for |
| --- | --- |
| Anas Aazib | Accounts, permissions, database, architecture |
| Abdullah Al-Jadani | Casting, team, tasks, film page |

Supervisor: Prof. Adeeb Rida Noor

## Current status

**CPIT-498: requirements and design.** This repository holds the documents and the designs. Application code starts in CPIT-499.

## Repository structure

```
docs/
  requirements/    Software Requirements Specification (ISO/IEC/IEEE 29148)
  design/          Software Design Document
  proposal/        Project proposal
  plan/            Weekly plan
    updates/       Weekly updates sent to the supervisor
  presentations/   Slides
design/
  diagrams/        Diagram sources (draw.io or PlantUML)
  exports/         Diagram images used in the documents
  wireframes/      Screen designs
```

In CPIT-499 the code is added under `apps/web` (front end) and `apps/api` (back end).

## Documents

| Document | Location |
| --- | --- |
| Software Requirements Specification | `docs/requirements/` |
| Software Design Document | `docs/design/` |
| Project proposal | `docs/proposal/` |
| Weekly plan and updates | `docs/plan/` |
| Presentations | `docs/presentations/` |

## Planned technology

| Part | Technology |
| --- | --- |
| Front end | Next.js, React, TypeScript |
| Back end | NestJS |
| Database | PostgreSQL |
| Film video | YouTube embedded player |
| Tests and deployment | GitHub Actions, Vercel |

## How we work

- Each member commits their own work, so the history shows who did what.
- One branch per task, merged into `main` through a pull request that the other member reviews.
- Commit messages are short and name the part they touch, for example `docs: add user stories for accounts (FR-ACC)`.
- Every Wednesday we send the supervisor a short update (done, blocked, next) and save a copy in `docs/plan/updates/`.

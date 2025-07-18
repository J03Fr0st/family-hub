# Family Tree Visualization Platform – Product Requirements Document (PRD)

## Document Control
| Item | Detail |
| --- | --- |
| **Version** | 0.2 (Updated) |
| **Author** | Joe Vreugdenburg |
| **Date** | 12 July 2025 |
| **Reviewed By** | — |
| **Status** | Draft |

---

## 1  Vision
Create an intuitive, web-based platform that lets families **explore, build, and maintain their genealogy** in a visually engaging tree. The application should feel instantly familiar to non-technical users while delivering enterprise-grade reliability, performance, and data integrity.

---

## 2  Problem Statement
Traditional genealogy tools are either overly complex, platform-locked, or lack real-time collaboration. Families need a **modern, browser-native solution** that:
* Shows complex relationships clearly (parents, spouses, adoptees, half-siblings, etc.).
* Allows quick edits without risking data corruption.
* Persists reliably in the cloud yet works offline when connectivity is poor.

---

## 3  Objectives & Success Criteria
| Objective | KPI / Target |
| --- | --- |
| **Fast visualisation** | Render 500-person tree in < 500 ms on average laptop |
| **Low friction CRUD** | Add/remove a person in ≤ 3 clicks |
| **High availability** | ≥ 99.5 % uptime (rolling 30 days) |
| **Cross-device UX** | Responsive layout for ≥ 320 px to 1920 px widths |
| **Data integrity** | Zero critical defects post-launch relating to relationship logic |

---

## 4  Scope
### In-Scope (MVP)
1. Browser UI built with Angular 19+ (standalone components)
2. REST API (Node.js 22 LTS + NestJS)
3. PostgreSQL 16+ persistence (Supabase managed)
4. Visual family tree (pan/zoom, collapsible branches)
5. CRUD for **Person** records & parent-child relationships
6. Auth via email/password (JWT) with role-based authorisation (Owner, Editor, Viewer)
7. Basic search/autocomplete by name
8. Export/backup to JSON

### Out-of-Scope (MVP)
* GEDCOM import/export (Phase 2)
* Real-time collaboration (Phase 2)
* Media uploads (photos, documents) (Phase 3)
* Mobile native app (post-MVP)

---

## 5  User Personas
| Persona | Description | Key Needs |
| --- | --- | --- |
| **Family Historian** | Mid-age hobbyist compiling multi-generation records | Rich editing, data accuracy, export |
| **Casual Relative** | Tech-savvy millennial checking lineage | Quick view, add self, edit photo |
| **Genealogy Researcher** | Professional compiling trees for clients | Bulk import/export, data validation |

---

## 6  User Stories (Excerpt)
1. **As an Owner**, I can **add a new person** with minimum details (name, gender, birth date) so that the tree updates instantly.
2. **As an Editor**, I can **establish a parent–child link** between two existing nodes to reflect lineage accurately.
3. **As a Viewer**, I can **search by name** to locate relatives in large trees within 1 second.
4. **As an Owner**, I can **delete a person** and reassign / confirm affected relationships to avoid orphaned nodes.
5. **As any user**, I can **pan, zoom, and collapse branches** to focus on a subtree without UI lag.

(The complete backlog is tracked separately in **Jira**; this section lists top priority items.)

---

## 7  Functional Requirements
| Ref ID | Requirement | Priority |
| --- | --- | --- |
| **FR-01** | System SHALL persist **Person** records in PostgreSQL with schema defined in §9.5. | Must |
| **FR-02** | System SHALL provide REST endpoints for Create, Read, Update, Delete (CRUD) of Person. | Must |
| **FR-03** | System SHALL support establishing & deleting **parent-child relationships** with referential integrity checks. | Must |
| **FR-04** | UI SHALL visualise the family tree using a scalable layout (d3-tree or ngx-graph) with smooth pan/zoom. | Must |
| **FR-05** | UI SHALL update visualisation **optimistically** on CRUD actions and reconcile with server result. | Should |
| **FR-06** | System SHALL provide full-text search on `firstName` + `lastName`. | Should |
| **FR-07** | System SHALL support **soft-delete** to allow undo within 30 minutes. | Could |
| **FR-08** | System SHALL log audit events (who, what, when) for all mutations. | Must |

---

## 8  Non-Functional Requirements (NFR)
* **Performance** – Core Web Vitals: LCP < 2.5s, FID < 100ms, CLS < 0.1
* **Scalability** – Architecture scales to 100k persons / organisation without rewrite
* **Security** – OWASP Top-10 compliant; passwords hashed (argon2) + HTTPS enforced
* **Accessibility** – WCAG 2.2 AA compliance with automated testing
* **Internationalisation** – UTF-8 everywhere; UI translatable via Angular i18n
* **Browser Support** – Last 2 major versions of Chrome, Edge, Firefox, Safari
* **SEO** – Server-side rendering support for public family trees

---

## 9  Technical Architecture
### 9.1  Front-End (Angular)
| Aspect | Choice |
| --- | --- |
| Language | TypeScript 5.7+ |
| Framework | Angular 19, Standalone Components |
| State | Angular Signals + NgRx v18 (hybrid approach) |
| UI Kit | Angular Material 19 + Tailwind CSS v4 |
| Graph | **ngx-graph** (d3 under the hood) with custom node templates |
| Testing | Vitest + Playwright (e2e) |

### 9.2  Back-End (Node)
* **Runtime:** Node.js 22 LTS
* **Framework:** NestJS 11+ (structured, DI-friendly)
* **ORM:** Drizzle ORM (lightweight, TypeScript-first)
* **Auth:** Lucia Auth (modern, secure)
* **Validation:** Zod (TypeScript-first validation)

### 9.3  Database (PostgreSQL)
* Version ≥ 16.x
* Deployment: Supabase (managed PostgreSQL) for dev/prod with built-in auth
* Indexes: `btree(last_name, first_name)`, `btree(parent_id)`, `btree(child_id)`
* Back-ups: Automated daily backups (7 days) + weekly (4 weeks)
* Real-time: PostgreSQL LISTEN/NOTIFY for real-time data synchronization

### 9.4  API (excerpt)
| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/api/person/:id` | Fetch person by UUID |
| POST | `/api/person` | Create new person |
| PUT | `/api/person/:id` | Update person |
| DELETE | `/api/person/:id` | Soft-delete person |
| POST | `/api/relationship` | Create parent-child relationship |
| DELETE | `/api/relationship/:id` | Remove parent-child relationship |
| GET | `/api/person/:id/ancestors` | Get all ancestors for a person |
| GET | `/api/person/:id/descendants` | Get all descendants for a person |

### 9.5  Data Model (PostgreSQL Schema)
```sql
-- Person table
CREATE TABLE person (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  first_name VARCHAR(255) NOT NULL,
  last_name VARCHAR(255) NOT NULL,
  middle_name VARCHAR(255),
  maiden_name VARCHAR(255),
  gender VARCHAR(10) CHECK (gender IN ('male', 'female', 'other')),
  birth_date DATE,
  death_date DATE,
  birth_place VARCHAR(255),
  death_place VARCHAR(255),
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE DEFAULT NULL
);

-- Parent-child relationships (biological focus)
CREATE TABLE relationship (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  parent_id UUID NOT NULL REFERENCES person(id) ON DELETE CASCADE,
  child_id UUID NOT NULL REFERENCES person(id) ON DELETE CASCADE,
  relationship_type VARCHAR(20) NOT NULL DEFAULT 'biological' 
    CHECK (relationship_type IN ('biological', 'adopted', 'step', 'foster')),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(parent_id, child_id),
  CHECK (parent_id != child_id)
);

-- Indexes for performance
CREATE INDEX idx_person_names ON person(first_name, last_name);
CREATE INDEX idx_person_active ON person(deleted_at) WHERE deleted_at IS NULL;
CREATE INDEX idx_relationship_parent ON relationship(parent_id);
CREATE INDEX idx_relationship_child ON relationship(child_id);
```

---

## 9.6 Project Structure & Organization

To ensure maintainability, scalability, and ease of onboarding, the project is organized as follows:

### 9.6.1 Monorepo Layout

The codebase uses a monorepo approach, with separate folders for frontend, backend, shared libraries, and infrastructure:

```
family-hub/
  apps/
    frontend/      # Angular 17+ app
    backend/       # NestJS 10+ API
  libs/
    shared/        # Shared TypeScript models, validation, utils
    ui/            # Reusable Angular UI components
  scripts/         # DevOps, migration, and utility scripts
  docs/            # Product docs, architecture, onboarding
  .github/         # GitHub Actions workflows, issue templates
  .env.example     # Example environment variables
  package.json     # Monorepo root scripts, dependencies
  README.md        # Project overview and quickstart
  ...
```

#### 9.6.2 Frontend (Angular)

- **apps/frontend/**
  - `src/app/`
    - `core/` – Singleton services (auth, API, state)
    - `features/` – Feature modules (tree, person, search, auth)
    - `shared/` – Shared pipes, directives, models
    - `ui/` – Design system components (from `libs/ui`)
    - `state/` – NgRx store, actions, reducers, effects
    - `assets/` – Images, translations, styles
    - `environments/` – Environment configs
  - `e2e/` – Cypress end-to-end tests

#### 9.6.3 Backend (NestJS)

- **apps/backend/**
  - `src/`
    - `modules/`
      - `person/` – Person entity, controller, service, DTOs
      - `relationship/` – Relationship logic, validation
      - `auth/` – Auth, roles, guards, JWT
      - `audit/` – Audit logging
    - `common/` – Shared interfaces, pipes, filters
    - `config/` – Environment and app config
    - `main.ts` – App bootstrap
    - `app.module.ts` – Root module
  - `test/` – Jest unit/integration tests

#### 9.6.4 Shared Libraries

- **libs/shared/**
  - TypeScript interfaces, validation schemas (Zod), utility functions
  - Used by both frontend and backend for type safety and DRYness

- **libs/ui/**
  - Angular Material-based components (buttons, dialogs, tree nodes)
  - Theming and accessibility helpers

#### 9.6.5 DevOps & Infrastructure

- **scripts/** – Automation (DB migrations, seeding, backups)
- **.github/** – CI/CD workflows (lint, test, build, deploy)
- **infra/** (optional) – Terraform, Docker, Kubernetes manifests

#### 9.6.6 Documentation

- **docs/** – PRD, architecture diagrams, API docs, onboarding guides
- **README.md** – Quickstart, contribution guide, architecture overview

#### 9.6.7 Best Practices

- **Testing:** All code must have unit (Jest), integration (Supertest), and e2e (Cypress) coverage. Test folders mirror source structure.
- **Code Quality:** Enforced via ESLint, Prettier, and Husky pre-commit hooks.
- **Environment:** All secrets/configs via `.env` files, never committed.
- **Onboarding:** `docs/ONBOARDING.md` provides step-by-step setup for new developers.

#### 9.6.8 Project Structure Reference Table

| Section   | Path             | Description                        |
|-----------|------------------|------------------------------------|
| Frontend  | `apps/frontend/` | Angular 17+ SPA, user interface    |
| Backend   | `apps/backend/`  | NestJS 10+ REST API                |
| Shared    | `libs/shared/`   | Types, validation, utils           |
| UI Library| `libs/ui/`       | Reusable Angular components        |
| Docs      | `docs/`          | Product, technical, onboarding docs|
| CI/CD     | `.github/`       | GitHub Actions workflows           |
| Scripts   | `scripts/`       | Automation scripts                 |

---

## 10  External Interfaces
* **Email service** (SendGrid) – account verification & password reset
* **CDN** (Cloudflare) – static assets & API caching
* **Telemetry** (OpenTelemetry + Jaeger) – tracing & metrics

---

## 10.5  Progressive Web App (PWA) Requirements
* **Service Worker** – Cache API responses and static assets for offline functionality
* **App Manifest** – Enable "Add to Home Screen" on mobile devices
* **Offline Mode** – Allow read-only access to cached family tree data when offline
* **Background Sync** – Queue changes made offline and sync when connection restored
* **Push Notifications** – Notify users of family tree updates (opt-in)

---

## 10.6  Privacy & Data Protection
* **GDPR Compliance** – Right to be forgotten, data portability, consent management
* **CCPA Compliance** – Data deletion requests, opt-out mechanisms
* **Data Minimization** – Collect only necessary personal information
* **Consent Management** – Granular permissions for data sharing within families
* **Data Retention** – Automatic deletion of inactive accounts after 2 years (with notice)
* **Cross-Border Data** – Ensure compliance with data residency requirements

---

## 10.7  Error Handling & User Feedback
* **Error Boundaries** – Graceful fallbacks for Angular component failures
* **Toast Notifications** – Non-intrusive success/error messages
* **Retry Mechanisms** – Automatic retry for failed API calls with exponential backoff
* **Offline Indicators** – Clear visual feedback when app is offline
* **Loading States** – Skeleton screens and progress indicators for all async operations
* **Validation Feedback** – Real-time form validation with helpful error messages

---

## 11  Security & Compliance
1. All traffic forced over HTTPS (TLS 1.3).
2. JWT access tokens (15 min) + refresh tokens (7 days).
3. Role-based access enforced at route guard (front-end) and NestJS guards (back-end).
4. Regular SCA (GitHub Dependabot) & SAST (CodeQL).

---

## 12  Performance & Scalability
* **Client-side render**: Virtualise nodes to avoid DOM bloat (ngx-cdk virtual scroll).
* **Server-side pagination**: API returns subtrees on demand for > 1k nodes.
* **Caching**: Redis layer for hot entities.

---

## 13  Analytics & Success Metrics
* Daily Active Users (DAU)
* Avg. tree nodes per user
* Time-to-first-render (TTFR)
* Error rate (< 0.1 % of requests)

---

## 14  Testing & QA Strategy
| Layer | Tool | Goal |
| --- | --- | --- |
| Unit | Vitest | 90% coverage front & back |
| Integration | Supertest (NestJS) | Validate API contracts |
| E2E | Playwright | Critical user journeys |
| Performance | Lighthouse CI | Core Web Vitals budgets |
| Security | OWASP ZAP + Snyk | Automated scans per build |
| Accessibility | axe-core | WCAG 2.2 AA compliance |

---

## 15  Accessibility & UX
* **Colour-blind safe palette**
* Keyboard navigation for all interactive elements
* ARIA labels on nodes; screen-reader friendly tree view

---

## 16  Deployment & DevOps
* **CI/CD**: GitHub Actions → Docker build → Push to GHCR
* **Infrastructure**: Terraform 1.10+ scripts provision VPC, ECS Fargate, Supabase PostgreSQL
* **Environments**: Dev, Staging, Prod (blue-green)
* **Observability**: OpenTelemetry + Prometheus + Grafana dashboards
* **Package Manager**: pnpm for faster installs and better dependency management
* **Build Tool**: esbuild for faster development builds

---

## 17  Assumptions & Dependencies
| Assumption | Impact if false |
| --- | --- |
| Supabase PostgreSQL is available in chosen region | Higher latency, possible data residency issues |
| Team familiar with Angular 19 and Drizzle ORM | Additional ramp-up time |

---

## 18  Risks & Mitigations
| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| Mis-handling complex relationships (e.g., adoption) | Medium | High | Comprehensive unit tests; validation rules |
| Visualisation performance degradation | Medium | Medium | Virtualisation + subtree loading |
| Data loss due to delete errors | Low | High | Soft-delete, daily backups, PostgreSQL ACID compliance |

---

## 19  Milestones & Timeline (Indicative)
| Phase | Duration | Deliverables |
| --- | --- | --- |
| **0 Discovery** | 1 week | Finalised PRD, high-level UX mocks |
| **1 Prototype** | 2 weeks | Clickable Angular prototype, dummy data |
| **2 MVP Build** | 4 weeks | CRUD API, PostgreSQL + Drizzle, auth, visual tree |
| **3 Beta** | 2 weeks | User feedback, perf tuning, a11y audit |
| **4 Launch** | 1 week | Prod deploy, marketing site, docs |

---

## 20  Future Enhancements
* GEDCOM import/export
* Real-time collaboration w/ WebSockets
* Media uploads & timeline stories
* AI-powered relationship suggestions

---

## 21  Open Issues / Questions
1. Do we support **multiple marriages** & complex unions in MVP?
2. Is offline-first (PWA + IndexedDB) required day 1?
3. Require SSO (Azure AD) for enterprise tier?
4. Should we implement real-time collaboration using WebSockets or Server-Sent Events?
5. Data migration strategy for existing family tree imports from other platforms?
6. Performance testing strategy for large family trees (10k+ members)?

---

## 22  Glossary
* **CRUD** – Create, Read, Update, Delete
* **JWT** – JSON Web Token
* **PWA** – Progressive Web App
* **WCAG** – Web Content Accessibility Guidelines

---

> **Next Steps:**
> * Collect stakeholder feedback on MVP scope (§4).
> * Finalise data model (§9.5) including relationship edge-cases.
> * UX team to produce high-fidelity mocks aligned with §14.

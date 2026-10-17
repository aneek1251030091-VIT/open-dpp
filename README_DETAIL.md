# open-dpp — Detailed Technical Documentation

> A technical reference for the `aneek1251030091-VIT/open-dpp` repository. This document describes the repository as implemented on the `main` branch and is intended for developers, maintainers, reviewers, and contributors.

## Index

1. [Project Summary](#1-project-summary)
2. [Technology Stack](#2-technology-stack)
3. [Repository Structure](#3-repository-structure)
4. [System Architecture](#4-system-architecture)
5. [Runtime Request and Data Flow](#5-runtime-request-and-data-flow)
6. [Backend Components](#6-backend-components)
7. [Frontend Components](#7-frontend-components)
8. [Shared Packages](#8-shared-packages)
9. [Core Algorithms and Design Patterns](#9-core-algorithms-and-design-patterns)
10. [Security and Authorization](#10-security-and-authorization)
11. [Persistence and Infrastructure](#11-persistence-and-infrastructure)
12. [AI and MCP Integration](#12-ai-and-mcp-integration)
13. [Validation and Error Handling](#13-validation-and-error-handling)
14. [Testing Strategy](#14-testing-strategy)
15. [Build, Deployment, and Operations](#15-build-deployment-and-operations)
16. [Environment Configuration](#16-environment-configuration)
17. [Important Design Considerations](#17-important-design-considerations)
18. [Useful Commands](#18-useful-commands)
19. [Reference Files](#19-reference-files)

---

## 1. Project Summary

`open-dpp` is an open-source platform for creating, managing, presenting, and integrating Digital Product Passports (DPPs). The implementation models Digital Product Passport data using the IEC 63278 Asset Administration Shell (AAS) concepts and exposes the functionality through a NestJS backend, a Vue 3 web application, typed shared packages, and automated tests.

The repository is a `pnpm` and Turborepo monorepo. The backend serves the API and, in production, serves the compiled frontend as static files. During development, the backend proxies non-API browser routes to the Vite development server.

### Main capabilities

- Manage Asset Administration Shells, submodels, concept descriptions, and AAS environments.
- Create and operate Digital Product Passports and their lifecycle statuses.
- Resolve product passports through unique product identifiers, permalinks, and GS1 resolver-style routes.
- Manage organizations, users, roles, invitations, API keys, authentication, and profile data.
- Import and export passport data, including bulk import and media/file handling.
- Define passport templates, presentation configurations, policies, and instance settings.
- Track activity history and domain changes.
- Provide analytics, status information, branding, email flows, and optional AI assistance.
- Integrate with S3-compatible object storage, MongoDB, email tooling, ClamAV, MCP, and external AI models.

---

## 2. Technology Stack

### 2.1 Languages and package management

| Area | Technology | Usage |
|---|---|---|
| Primary language | TypeScript | Backend, frontend, shared packages, DTOs, tests, and tooling |
| Frontend markup | Vue Single File Components | UI screens, layouts, and reusable components |
| Styling | CSS, Tailwind CSS v4, PrimeVue theme tokens | Application styling and component design |
| Shell | Bash/sh | Setup scripts, Docker startup, and Make targets |
| Package manager | pnpm 11 | Workspace dependency management and package scripts |
| Monorepo orchestration | Turborepo | Builds, development tasks, tests, and task dependency scheduling |

### 2.2 Backend

- **NestJS 11** for dependency injection, modules, HTTP controllers, middleware, guards, filters, and WebSocket gateways.
- **Express 5** as the HTTP platform used by NestJS.
- **Mongoose 9 / MongoDB 7** for document persistence.
- **Zod** for schemas, parsing, DTO validation, environment validation, and domain boundary checks.
- **Better Auth** for authentication, API keys, and OAuth provider capabilities.
- **CASL (`@casl/ability`)** for permission evaluation.
- **Socket.IO** for real-time AI chat communication.
- **Swagger / OpenAPI** for API documentation generation.
- **MinIO-compatible client** for S3-compatible object storage; local deployment uses RustFS.
- **Nodemailer, MJML, and Handlebars** for email composition and delivery.
- **LangChain, LangGraph, Mistral integrations, and MCP SDKs** for AI agents and tool use.
- **`xlsx`, `archiver`, `sharp`, `file-type`, and `multer`** for import, export, image, file, and upload processing.

### 2.3 Frontend

- **Vue 3** with TypeScript and `<script setup>` components.
- **Vite 7** for development and production bundling.
- **Vue Router 5** for navigation, route metadata, and guards.
- **Pinia 3** for application state management.
- **PrimeVue 4**, PrimeIcons, and the Aura theme preset for UI components.
- **Tailwind CSS 4** for utility styling.
- **Axios** through the shared API client for HTTP requests.
- **Vue I18n 11** for localization and language persistence.
- **Vee Validate + Zod** for form validation.
- **Chart.js + Vue Chart.js** for analytics visualizations.
- **Socket.IO client** for real-time AI interactions.
- **DOMPurify and Marked** for safe rendering of formatted content.
- **Day.js** for date and locale formatting.

### 2.4 Testing and documentation

- **Jest and SWC** for backend and shared-package unit/integration tests.
- **Vitest and Testing Library** for frontend tests.
- **Playwright** for end-to-end browser tests.
- **MongoMemoryReplSet** for MongoDB integration tests that require transactions.
- **VitePress** for repository documentation.
- **Changesets** for versioning and package release management.
- **Oxlint and Oxfmt** for linting and formatting.

---

## 3. Repository Structure

```text
open-dpp/
├── apps/
│   ├── main/                       NestJS backend and production HTTP server
│   ├── client/                     Vue 3 + Vite frontend
│   └── e2e/                        Playwright end-to-end test project
├── packages/
│   ├── dto/                        Shared Zod DTO schemas and TypeScript types
│   ├── api-client/                 Typed Axios API client used by the frontend
│   ├── env/                        Environment schema, EnvModule, and EnvService
│   ├── exception/                  Domain errors, filters, and validation helpers
│   ├── permission/                 CASL permission definitions and helpers
│   └── testing/                    Shared test factories and testing utilities
├── docs/                           VitePress documentation and generated API docs
├── scripts/                        Setup and code-generation utilities
├── docker/                         Docker startup and database support files
├── .github/                        CI, automation, and repository configuration
├── docker-compose.yml              Example full-stack deployment
├── docker-compose.dev.yml          Local development infrastructure
├── Makefile                        Docker lifecycle and cleanup commands
├── package.json                    Root pnpm scripts and development dependencies
├── pnpm-workspace.yaml             Workspace package definitions and install policy
├── turbo.json                      Turborepo task graph and caching configuration
├── Dockerfile                      Multi-stage production image build
├── CLAUDE.md                       Repository development guidance
├── CONTEXT.md                      Domain language glossary
└── README_DETAIL.md                This detailed technical reference
```

### Backend feature structure

Most backend bounded contexts follow a domain-oriented structure:

```text
feature/
├── domain/                         Entities, value objects, invariants, visitors
├── application/                    Use-case orchestration
├── infrastructure/                 Mongoose repositories, schemas, integrations
└── presentation/                   Controllers, gateways, DTO adapters
```

The actual feature modules include `aas`, `digital-product-document`, `passports`, `identity`, `policy`, `media`, `templates`, `traceability-events`, `analytics`, `ai`, and other supporting modules.

---

## 4. System Architecture

### 4.1 High-level architecture

```text
┌────────────────────────────────────────────────────────────────────┐
│                        Browser / External Client                   │
│  Vue 3 UI · API client · Socket.IO · GS1/resolver URLs             │
└───────────────────────────────┬────────────────────────────────────┘
                                │ HTTP / WebSocket
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│                         NestJS Main Application                    │
│                                                                    │
│  Middleware → Versioning → AuthGuard → PolicyGuard → Controller    │
│                                  │                                 │
│                  Application services / domain entities            │
│                                  │                                 │
│       Mongoose repositories · S3 media · mail · AI/MCP tools       │
└───────────────┬─────────────────────┬──────────────────┬───────────┘
                │                     │                  │
                ▼                     ▼                  ▼
        MongoDB replica set       RustFS/S3          Mailpit/SMTP
        users, passports,         media and files     local email UI
        AAS data, policies

                └──────────────► Optional ClamAV scanning

        AI provider / Mistral ◄── LangChain agent ◄── MCP servers/tools
```

### 4.2 Backend module composition

`apps/main/src/app.module.ts` composes the application from independent NestJS modules. Important modules include:

- `AasModule`: AAS domain, submodels, elements, environment operations, and AAS endpoints.
- `UniqueProductIdentifierModule`: Product identifier generation and resolution.
- `BrandingModule`: Instance branding and public presentation assets.
- `BulkImportModule`: Batch import workflows.
- `TemplatesModule`: Passport and submodel templates.
- `PassportsModule`: Digital Product Passport lifecycle and management.
- `OrganizationsModule`, `UsersModule`, `ApiKeysModule`, and `AuthModule`: Identity and access management.
- `EmailChangeRequestsModule`: Verified email change workflow.
- `TraceabilityEventsModule`: Product traceability event data.
- `MediaModule`: Upload, storage, retrieval, and optional virus scanning.
- `AiModule` and `AiConfigurationModule`: AI configuration, agent execution, and chat.
- `McpServerModule`: Model Context Protocol server/client integration.
- `AnalyticsModule`: Usage and product analytics.
- `PolicyModule`: Policy evaluation, quotas, and access decisions.
- `PresentationConfigurationsModule`: Public and controlled passport presentations.
- `PermalinkModule`: Stable links that resolve to passports.
- `StatusModule` and `InstanceSettingsModule`: Health/status and instance-level behavior.

### 4.3 Production serving model

The Docker image builds both the backend and frontend. The production image copies:

- backend output to `/app/dist`;
- frontend output to `/app/dist/client/dist`;
- public assets to `/app/public`;
- only the production dependencies of the main application.

NestJS uses `ServeStaticModule` to serve the compiled Vue application and reserves `/api` routes for backend APIs. In development, `main.ts` proxies non-API routes to `http://localhost:5173` so the backend remains the single local entry point.

---

## 5. Runtime Request and Data Flow

### 5.1 Standard API request

1. A browser or external client calls `/api/v1/...`.
2. Correlation-ID and logging middleware execute.
3. NestJS applies authentication and policy guards.
4. The controller validates route parameters and request bodies using DTO/Zod rules.
5. The application service loads domain entities through repository abstractions.
6. Domain methods apply invariants, permissions, lifecycle changes, and change tracking.
7. Repositories persist the resulting domain state through Mongoose.
8. The controller maps the result to a shared DTO response.
9. Exception filters translate domain and validation errors into API responses.

### 5.2 Frontend startup flow

`apps/client/src/main.ts`:

1. Creates the Vue application and Pinia instance.
2. Installs Vue I18n, PrimeVue, confirmation and toast services.
3. Reads the Better Auth session.
4. Loads the current user through `apiClient.dpp.users.getMe()` when signed in.
5. Applies the user's preferred language.
6. Loads organizations and selects the active organization from the session or the first available organization.
7. Installs the router and mounts the application.

### 5.3 Public passport resolution

The backend recognizes GS1 resolver-style routes before applying the `/api` global prefix. The route matcher supports combinations of:

- GTIN (`01`);
- batch/lot (`10`);
- serial number (`21`).

Permalinks use a similar resolution path: the permalink service resolves a public link to a passport, then downstream services load the associated unique product identifier and passport.

---

## 6. Backend Components

### 6.1 Asset Administration Shell and AAS domain

The AAS domain represents IEC 63278-style structures, including:

- Asset Administration Shells;
- Asset Information;
- Submodels;
- Concept Descriptions;
- References and Keys;
- Properties and Ranges;
- Files and Blobs;
- Multi-language Properties;
- Entities;
- Relationship Elements;
- Submodel Element Collections and Lists;
- Extensions, qualifiers, resources, and security metadata.

AAS objects expose `fromPlain`, `loadFromDb`, `toPlain`, and `accept(visitor)` patterns. This keeps domain objects separate from transport and persistence representations.

### 6.2 Digital Product Documents and passports

The DPP/passport components combine AAS environments with product-specific lifecycle, presentation, identifier, policy, and traceability concerns. A passport can be:

- created and edited;
- associated with a unique product identifier;
- published for presentation;
- archived;
- restored from an archive state;
- accessed through a permalink or resolver route;
- presented with policy-based filtering;
- enriched with media and traceability events.

### 6.3 Identity and organization management

Identity functionality covers:

- sign-in and sign-up;
- sessions and authentication accounts;
- users and profiles;
- preferred language;
- organization membership;
- owner/member roles;
- invitations and membership removal;
- API keys;
- email change requests;
- optional OAuth provider behavior;
- user banning and related account state.

The domain glossary in `CONTEXT.md` defines important distinctions such as User versus Account, Member versus Organization, and email-change authorization versus verification token.

### 6.4 Policy and permissions

The policy module enforces instance or organization rules such as AI token quotas and feature availability. AAS permissions use CASL-based abilities and path-aware checks so permissions can be applied to nested AAS elements.

### 6.5 Media and object storage

Media uploads pass through the backend rather than being anonymously exposed from object storage. RustFS provides S3-compatible storage in local Docker environments. Buckets are initialized with object versioning for DPP data and profile pictures. ClamAV integration is optional and controlled through `OPEN_DPP_CLAMAV_URL`.

### 6.6 Templates and presentation configurations

Templates provide reusable structures for creating passports or submodels. Presentation configurations define how passport data is selected, transformed, and exposed to viewers. The same permission-aware serialization approach used by AAS visitors supports controlled public presentation.

### 6.7 Import/export and data conversion

The backend includes bulk import and export support. Dependencies such as `xlsx`, `archiver`, `jsonata`, and file-type detection support spreadsheet import, archive generation, transformation, and safe file handling.

### 6.8 Traceability and activity history

Traceability events represent product history. Activity history records domain changes such as:

- submodel added to an environment;
- submodel deleted or moved;
- property value changed;
- file value changed;
- reference element value changed;
- display name or description changed;
- thumbnails modified.

The visitor and tracking patterns allow changes to be recorded close to the domain operation that caused them.

---

## 7. Frontend Components

### 7.1 Application shell and layouts

`App.vue` selects one of three lazy-loaded layouts based on route metadata:

- `Main.vue`: default authenticated application layout;
- `Presentation.vue`: public or presentation-focused layout;
- `None.vue`: pages such as sign-in and password reset that do not use the main shell.

The application also provides global toast and confirmation UI.

### 7.2 Routing

The router groups routes into authentication, organization, media, presentation, and administration route collections. It performs:

- session-aware authentication redirects;
- anonymous-only route handling;
- organization parameter validation;
- active organization synchronization;
- page loading state management;
- scroll preservation for query-only navigation;
- lazy loading of view components.

### 7.3 State management

Pinia stores hold cross-view state, including:

- current user and session;
- organization list and active organization;
- selected passport;
- media state;
- AI agent state;
- analytics;
- status and HTTP error state;
- presentation configurations;
- language and layout preferences;
- notifications.

The stores call the typed API client instead of directly constructing raw HTTP requests.

### 7.4 UI and forms

PrimeVue provides accessible UI primitives, dialogs, menus, tables, forms, and notifications. Tailwind CSS provides utility classes. Vee Validate and Zod provide form validation consistent with backend schemas. Vue I18n manages localized labels and messages.

### 7.5 Analytics and presentation

Chart.js renders analytics data. Public presentation views use dedicated routes and layouts so a passport can be viewed without exposing the full administrative UI. `DOMPurify` is used where user- or model-generated HTML must be rendered safely.

---

## 8. Shared Packages

### `@open-dpp/dto`

The DTO package is the contract layer shared by backend and frontend. It contains Zod schemas, DTO types, enums, API version definitions, AAS schemas, passport schemas, policy schemas, organization/user schemas, and presentation schemas.

### `@open-dpp/api-client`

This package exposes a typed Axios client. It depends on `@open-dpp/dto`, so frontend calls and response types are aligned with backend contracts.

### `@open-dpp/env`

Defines the environment schema and provides `EnvModule` and `EnvService` for validated configuration access inside NestJS.

### `@open-dpp/exception`

Centralizes domain and service error types, exception handlers, HTTP filters, WebSocket filters, Zod validation helpers, and parsing utilities. Important domain errors include `ValueError`, `NotFoundError`, `NotSupportedError`, and forbidden-access errors.

### `@open-dpp/permission`

Provides CASL-based permission primitives used by policy and AAS security checks.

### `@open-dpp/testing`

Provides reusable test factories, including Fishery-based data factories used by backend, frontend, and API client tests.

---

## 9. Core Algorithms and Design Patterns

### 9.1 Domain-driven immutable entities

Entities such as `User` use private constructors and factory methods:

- `create()` creates new domain instances;
- `loadFromDb()` reconstructs persisted instances;
- `fromPlain()` validates external or serialized input;
- `withX()` returns a new instance with a changed value;
- `toPlain()` serializes the domain model.

For example, `User.withRole()` and `User.withPreferredLanguage()` return a new `User` rather than mutating the original object. This keeps state transitions explicit and testable.

### 9.2 Repository-based persistence

Application services do not call Mongoose models directly. The repository layer loads domain objects, while services invoke domain operations and persist the resulting entity. This prevents database update syntax from bypassing domain invariants.

The intended update algorithm is:

```text
load entity from repository
        ↓
apply domain method / withX method
        ↓
validate invariants and permissions
        ↓
persist the resulting entity through repository.update/save
        ↓
return DTO representation
```

### 9.3 Visitor pattern for AAS trees

AAS structures are recursive trees containing many element types. `IVisitor<ContextT, R>` defines one visitor method per element type. Domain objects call `accept(visitor, context)`.

This enables multiple operations without putting every operation into every domain class:

- `JsonVisitor`: recursively converts AAS objects to plain JSON.
- `ModifierVisitor`: validates and applies supported modifications.
- `ValueModifierVisitor`: changes values within the tree.
- `ValueVisitor`: extracts values from the tree.

This is particularly useful for nested `SubmodelElementCollection` and `SubmodelElementList` structures.

### 9.4 Permission-aware serialization

`JsonVisitor` checks the AAS ability before returning an element. The element path is derived through `getIdShortPath()` and checked with `ability.can(Permissions.Read, path)`. Elements that the subject cannot read are removed or serialized as empty results. Submodel references can also be filtered to match a selected set of submodels.

This algorithm allows the same domain tree to produce different representations for different users or presentation configurations.

### 9.5 Permission-aware modification

`ModifierVisitor` follows this process:

1. Parse the modification payload with the relevant DTO schema.
2. Derive the target element's `idShort` path.
3. Check `Permissions.Edit` through the supplied `AasAbility`.
4. Apply the supported modification.
5. Validate language uniqueness and other domain rules.
6. Record a typed activity-history change event.
7. Reject unsupported element types with `NotSupportedError`.

Nested modifications locate child elements by `idShort` and recursively call `accept()`.

### 9.6 Environment hydration algorithm

`Environment` stores only IDs for shells, submodels, and concept descriptions. `ExpandedEnvironment.fromEnvironment()` hydrates these references using maps:

1. Check every referenced ID against the corresponding map.
2. Collect all missing shell, submodel, and concept-description IDs.
3. Throw one domain-friendly `ValueError` if any references are missing.
4. Preserve the environment's ordering while replacing IDs with domain objects.

This separates lightweight persistence references from fully loaded domain graphs.

### 9.7 DPP status transition algorithm

The DPP lifecycle supports draft, published, and archived states. Transitions are guarded explicitly:

```text
Draft ──publish──► Published
Draft/Published ──archive──► Archived
Archived ──restore──► previous non-archived status
```

Invalid transitions produce `ValueError`, for example publishing a non-draft document, archiving an already archived document, or restoring an item with no valid previous status.

### 9.8 AI quota enforcement algorithm

The AI request path in `ChatService.askAgent()` is:

1. Resolve the unique product identifier.
2. Load the referenced passport.
3. Enforce the organization's AI token quota before processing.
4. Load the organization's AI configuration.
5. Reject the request if AI is disabled.
6. Build the configured LLM and MCP tools.
7. Create the LangChain agent and prompt it with the passport ID.
8. Execute the chain.
9. Read token usage metadata from the LLM response.
10. Increment the organization's quota by the total token count.

This prevents expensive AI execution after a quota has already been exceeded.

### 9.9 Organization selection algorithm

At frontend startup, the active organization is selected in this order:

1. Use the active organization ID from the authenticated session if available.
2. Otherwise select the first organization returned by the API.
3. If the user has no organizations, select `null` and route the user toward organization creation.

---

## 10. Security and Authorization

Security is implemented in multiple layers:

- Better Auth manages authentication sessions and account flows.
- `AuthGuard` is registered as a global NestJS guard.
- `PolicyGuard` is registered globally for policy enforcement.
- WebSocket endpoints use `WebsocketAuthGuard`.
- AAS abilities evaluate read, edit, and delete permissions at element paths.
- Public routes are explicitly separated from authenticated API routes.
- Media downloads are proxied by the backend rather than exposed anonymously from RustFS.
- Upload scanning is optional through ClamAV integration.
- Zod schemas validate request, environment, and serialized data at boundaries.
- CORS is enabled with credentials for the local frontend origin.
- API versioning is enabled using URI version prefixes.

Production deployments should replace example secrets, configure authenticated database access, use HTTPS, and configure secure object-storage credentials.

---

## 11. Persistence and Infrastructure

### MongoDB

MongoDB runs as a replica set, including in local development, so transactions can be exercised by the application and integration tests. The backend uses Mongoose configuration generated from `EnvService`.

### RustFS / S3-compatible storage

RustFS is used as the local S3-compatible object store. The Compose initialization service creates:

- the default DPP bucket;
- the profile-picture bucket;
- object versioning for both buckets.

### Mailpit

Mailpit captures local emails and provides a web interface on port `8025`. This supports password, invitation, email verification, and email-change flows without sending real mail during development.

### Optional ClamAV

If `OPEN_DPP_CLAMAV_URL` is unset, the backend logs a warning and uploaded files are not virus-scanned. Production deployments can add a ClamAV REST service and configure the URL.

---

## 12. AI and MCP Integration

The AI subsystem is built around LangChain and MCP:

- `AiConfigurationModule` stores organization-level provider/model configuration.
- `AiService` creates the configured language model and agent.
- `McpClientService` discovers tools from configured MCP servers.
- `ChatService` resolves a passport, checks policy quota, creates the chain, and records token usage.
- `ChatGateway` exposes the chat interaction over Socket.IO at an API-versioned WebSocket path.
- `@modelcontextprotocol/sdk` and LangChain MCP adapters support tool interoperability.
- Mistral is configured through `OPEN_DPP_MISTRAL_API_KEY`.

The WebSocket message flow is:

```text
client emits userMessage
        ↓
resolve permalink to passport
        ↓
find unique product identifier
        ↓
authenticate optional user/member context
        ↓
enforce organization AI quota
        ↓
load AI configuration and MCP tools
        ↓
run LangChain agent
        ↓
emit botMessage or limitError
```

---

## 13. Validation and Error Handling

### Validation

- Environment variables are parsed by the shared environment package.
- DTOs use shared Zod schemas.
- AAS plain data is parsed before domain reconstruction.
- Modification payloads are parsed before applying changes.
- Global NestJS `ValidationPipe` handles framework-level validation.
- Invalid serialized environments and domain records are converted into `ValueError` with contextual messages.

### Error hierarchy

The repository uses domain-oriented errors instead of leaking persistence or framework exceptions into business logic:

- `ValueError`: invalid domain state or operation.
- `NotFoundError`: requested entity does not exist.
- `NotSupportedError`: operation is not implemented for an element type.
- Forbidden errors: subject lacks permission for an operation.
- Quota errors: organization has exceeded an AI or policy limit.

Global filters in `main.ts` map errors to HTTP responses. Socket.IO errors use `SocketIoExceptionFilter`.

---

## 14. Testing Strategy

### Unit tests

Jest tests cover domain entities, status transitions, visitors, stores, utility functions, and shared packages. Backend tests run with SWC and ECMAScript module support.

### Integration tests

Backend integration tests use MongoDB memory replica sets where transactions and repository behavior must be exercised against a real MongoDB-compatible engine.

### Frontend tests

Vitest and Testing Library cover Vue components, Pinia stores, router guards, error handling, and API interactions. MSW is used in shared/API-client tests where request mocking is appropriate.

### End-to-end tests

Playwright tests in `apps/e2e` exercise browser-level flows against the running application and supporting Docker services.

### Test infrastructure

`make test` starts the test Docker services. Turborepo runs package-specific test tasks without caching test results by default, as configured in `turbo.json`.

---

## 15. Build, Deployment, and Operations

### Development deployment

`docker-compose.dev.yml` starts MongoDB, RustFS, RustFS bucket initialization, and Mailpit. The application itself is normally started from pnpm so that NestJS and Vite watch mode are available.

### Example full deployment

`docker-compose.yml` starts the prebuilt `ghcr.io/open-dpp/open-dpp` image together with MongoDB, RustFS, RustFS initialization, and Mailpit. The file is explicitly marked as an example deployment and not a production configuration.

### Production image

The `Dockerfile` uses a two-stage build:

1. Install workspace dependencies with pnpm.
2. Build all required packages and applications.
3. Deploy only production dependencies for `@open-dpp/main`.
4. Copy backend, frontend, public assets, and startup script into a smaller runtime image.
5. Start the backend on port `3000`.

### Turborepo task behavior

- `build` depends on upstream package builds and produces `dist/**`.
- `docs:build` produces VitePress output.
- `dev` is persistent and not cached.
- `test` and `e2e-test` are not cached and include `.env` as an input.
- `check-ts` propagates through workspace dependencies.

---

## 16. Environment Configuration

Copy the appropriate example file before running the system:

```bash
cp .env.dev.example .env.dev
cp .env.example .env
```

Important configuration groups include:

| Group | Variables / purpose |
|---|---|
| Application | `OPEN_DPP_PORT`, `OPEN_DPP_URL`, branding, permalink base URL |
| MongoDB | `OPEN_DPP_MONGODB_URI` and optional host/port/user/password/database values |
| AI | `OPEN_DPP_MISTRAL_API_KEY` |
| S3/RustFS | endpoint, port, SSL flag, access key, secret, DPP bucket, profile bucket |
| ClamAV | optional `OPEN_DPP_CLAMAV_URL` |
| Mail | SMTP host, port, credentials, sender address |
| Authentication | `OPEN_DPP_AUTH_SECRET` |
| OAuth provider | optional enabled flag, client ID/secret, redirect URIs, client name |
| Instance settings | signup and organization-creation feature switches |

Do not commit real secrets. The example Docker configuration contains development-oriented defaults and must be hardened for production.

---

## 17. Important Design Considerations

1. **Preserve domain invariants.** Load an entity, call its domain method, and persist the resulting state. Avoid direct field-level database updates for domain state.
2. **Keep DTOs shared.** Add request and response schemas to `@open-dpp/dto` when both frontend and backend need the contract.
3. **Use AAS visitors for tree operations.** Avoid duplicating recursive traversal logic in controllers or services.
4. **Apply permission checks at the correct path.** AAS authorization is element- and `idShort`-path-aware.
5. **Keep `Environment` and `ExpandedEnvironment` separate.** The former stores references; the latter contains hydrated domain objects.
6. **Record domain changes at the mutation point.** Activity history depends on typed change events created by domain operations and visitors.
7. **Treat public presentations as filtered projections.** Never assume the full passport graph is safe to expose.
8. **Use transactions where multiple MongoDB documents must remain consistent.** Local Compose intentionally runs MongoDB as a replica set for this purpose.
9. **Check AI quota before invoking the model.** Token usage is incremented after successful model usage metadata is available.
10. **Use versioned APIs.** New endpoints should follow the existing URI versioning and shared API-version DTO conventions.

---

## 18. Useful Commands

### Install dependencies

```bash
pnpm install
```

### Prepare local infrastructure

```bash
cp .env.dev.example .env.dev
make dev
```

### Run all applications

```bash
pnpm run dev
```

### Run only the backend

```bash
pnpm run dev:main
```

### Build

```bash
pnpm run build
pnpm run build:main
pnpm run build:docs
```

### Lint and format

```bash
pnpm run lint
pnpm run lint:fix
pnpm run format
pnpm run format:check
```

### Test

```bash
make test
pnpm run test
pnpm run test:main
pnpm run test:client
pnpm run test:e2e
```

### Run one backend test

```bash
cd apps/main
NODE_OPTIONS=--experimental-vm-modules pnpm exec jest path/to/file.spec.ts
```

### Run one frontend test

```bash
cd apps/client
pnpm exec vitest run src/path/to/file.spec.ts
```

### Generate API documentation

```bash
cd apps/main
pnpm run export-api-doc
```

### Start the example full stack

```bash
cp .env.example .env
make full
```

---

## 19. Reference Files

The following files are the main implementation references for this document:

- `README.md` — project introduction and getting-started links.
- `CLAUDE.md` — development commands, architecture guidance, and domain patterns.
- `CONTEXT.md` — identity, organization, and account-management terminology.
- `package.json` — workspace scripts and core tooling.
- `pnpm-workspace.yaml` — workspace membership and dependency installation policy.
- `turbo.json` — build, development, documentation, and test task graph.
- `apps/main/src/main.ts` — NestJS bootstrap, proxying, API versioning, global filters, and server startup.
- `apps/main/src/app.module.ts` — backend module composition and global guards.
- `apps/main/src/aas/domain/visitor.ts` — AAS visitor interface.
- `apps/main/src/aas/domain/json-visitor.ts` — permission-aware AAS serialization.
- `apps/main/src/aas/domain/modifier-visitor.ts` — validated AAS modification and change tracking.
- `apps/main/src/aas/domain/environment.ts` — reference-based environment model and mutations.
- `apps/main/src/aas/domain/expanded-environment.ts` — environment hydration and reconstruction.
- `apps/main/src/digital-product-document/domain/digital-product-document-status.ts` — DPP lifecycle transitions.
- `apps/main/src/identity/users/domain/user.ts` — immutable user entity pattern.
- `apps/main/src/ai/chat.service.ts` — AI agent orchestration and quota accounting.
- `apps/main/src/ai/presentation/chat.gateway.ts` — Socket.IO AI chat interface.
- `apps/client/src/main.ts` — frontend bootstrapping and initial session/organization loading.
- `apps/client/src/App.vue` — layout selection and global UI services.
- `apps/client/src/router/index.ts` — routes, guards, organization selection, and scroll behavior.
- `Dockerfile` — multi-stage production image.
- `docker-compose.dev.yml` — local MongoDB, RustFS, and Mailpit services.
- `docker-compose.yml` — example full-stack deployment.
- `.env.example` — environment variable reference.

---

*Generated from repository structure and source files in `aneek1251030091-VIT/open-dpp` on the `main` branch.*

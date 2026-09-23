# OfficeDepot_Honduros Engineering Constitution
**Version:** 1.0.0

> **Stated Assumption:** As the project brief, maturity, shape, and primary languages were delegated, this constitution assumes a **production-grade full-stack e-commerce web application** tailored for Office Depot's regional operations in Honduras. The stack is inferred as TypeScript, Next.js (Frontend), and Node.js/Express (Backend) to support robust B2B/B2C retail operations. This document serves as the **supreme engineering authority** for the project.

## Mission
To provide a seamless, reliable, and localized e-commerce and inventory management platform for Office Depot customers and operators in Honduras, ensuring high availability, secure transactions, and accurate regional compliance.

## Core Values
1. **Testing over trust:** We do not assume code works; we prove it through rigorous, automated testing.
2. **Reliability at scale:** E-commerce operations demand high availability. We design for failure and degrade gracefully.
3. **Security by default:** Customer data, payment information, and business intelligence are protected at every layer.

## Technology Stack

### Required Technologies
| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Language** | TypeScript (Strict Mode) | End-to-end type safety |
| **Frontend** | Next.js / React | UI rendering, routing, and server-side rendering |
| **Styling** | Tailwind CSS | Utility-first responsive design |
| **Backend** | Node.js / Express | API gateway and business logic |
| **Database** | PostgreSQL | Primary relational data store |
| **ORM** | Prisma | Database access and schema management |
| **Validation** | Zod | Runtime schema validation |

### Forbidden Technologies / Practices
* Plain JavaScript in application code (TypeScript is mandatory).
* Unmaintained dependencies (libraries without updates in the last 12 months).
* Experimental libraries in production without explicit architectural approval.
* Direct database queries from the frontend client.

## Repository Structure
The project utilizes a Turborepo monorepo structure to share types and UI components between the frontend and backend.

```text
office-depot-honduros/
├── apps/
│   ├── web/                # Next.js frontend application
│   └── api/                # Node.js/Express backend services
├── packages/
│   ├── ui/                 # Shared React components
│   ├── config/             # Shared ESLint, TSConfig, Prettier
│   └── database/           # Prisma schema and client
├── docs/                   # Architecture and API documentation
└── .github/                # CI/CD workflows
```

## Language/Code Standards
* **Typing:** `strict: true` must be enabled in all `tsconfig.json` files. `any` is strictly forbidden; use `unknown` if the type is truly dynamic.
* **Naming Conventions:**
  * Variables, functions, methods: `camelCase`
  * Classes, Interfaces, Types, React Components: `PascalCase`
  * Files and directories: `kebab-case` (e.g., `order-controller.ts`, `shopping-cart.tsx`)
  * Constants and Environment Variables: `UPPER_SNAKE_CASE`
* **Size Limits:** Files should target **300 lines maximum**. Any file exceeding **500 lines** triggers a mandatory refactor blocking the PR.

## Frontend Standards
* **State Management:** Use React Context for global UI state; use React Query (TanStack Query) for server state and data fetching.
* **Component Architecture:** Favor Server Components where interactivity is not required. Client components must be explicitly marked with `"use client"`.
* **Data Fetching:** All API calls must be strongly typed using shared interfaces from the backend.

## Backend/API & Validation Standards
* **Architecture:** RESTful API design. Controllers handle HTTP concerns, Services handle business logic, Repositories/Prisma handle data access.
* **Validation:** All incoming requests (body, query, params) MUST be validated at the boundary using Zod schemas.
* **Response Format:** Standardized JSON responses:
  ```json
  {
    "success": true|false,
    "data": { ... },
    "error": { "code": "...", "message": "..." } // Only if success is false
  }
  ```

## Error Handling
Errors must be categorized using the following standard taxonomy. Raw exceptions must never be leaked to the client.

| Category | HTTP Status | Description |
| :--- | :--- | :--- |
| `VALIDATION_ERROR` | 400 | Invalid input data (e.g., malformed email, missing fields). |
| `AUTHENTICATION_ERROR` | 401 | Missing or invalid credentials. |
| `AUTHORIZATION_ERROR` | 403 | Authenticated user lacks required permissions. |
| `BUSINESS_ERROR` | 422 | Rule violation (e.g., insufficient inventory). |
| `EXTERNAL_SERVICE_ERROR` | 502 | Failure in a third-party integration (e.g., payment gateway). |
| `INFRASTRUCTURE_ERROR` | 500 | Database or cache connectivity issues. |
| `UNKNOWN_ERROR` | 500 | Unhandled exceptions. |

## Logging
All logs must be structured as JSON. `console.log` is forbidden in production; use the designated logging library (e.g., Winston or Pino).

**Required Structured Log Fields:**
* `event`: String identifier for the action (e.g., `order_placed`).
* `timestamp`: ISO 8601 UTC datetime.
* `requestId`: UUID for distributed tracing.
* `userId`: (Optional) ID of the authenticated user.
* `metadata`: (Optional) Contextual JSON payload (must not contain PII/PCI data).

## Security
* **Authentication & Authorization:** Must occur server-side. The frontend is considered an untrusted client.
* **Secrets Handling:**
  * Must be loaded from environment variables at runtime.
  * Production secrets must be provisioned via a Secret Manager (e.g., AWS Secrets Manager).
  * **Never commit secrets** to version control. Pre-commit hooks must scan for entropy/secrets.
* **Data Protection:** Credit card data must never touch our servers directly (use tokenized payment gateways like Stripe/CyberSource).

## Accessibility
* All frontend components must meet WCAG 2.1 AA standards.
* Semantic HTML is required (e.g., `<button>` for actions, `<a>` for navigation).
* `aria-` attributes must be used where semantic HTML is insufficient.
* Automated accessibility testing (e.g., `axe-core`) must run in CI.

## Performance
* **Frontend:** Core Web Vitals must be maintained (LCP < 2.5s, FID < 100ms, CLS < 0.1).
* **Backend:** API endpoints must respond in < 200ms at the 95th percentile.
* **Database:** Queries scanning more than 10,000 rows must be indexed. N+1 query patterns are forbidden and must be caught in code review.

## Testing
Testing is our primary core value. Code without tests is considered incomplete.

* **Minimum Coverage:** 80% minimum overall; **95% for critical business logic** (pricing, cart, taxes).
* **Required Test Types:**
  * **Unit Tests:** (Jest) For utilities, services, and complex UI components.
  * **Integration Tests:** (Jest/Supertest) For API endpoints and database queries.
  * **E2E Tests:** (Cypress/Playwright) Mandatory for the following critical paths:
    * Login / Authentication
    * Checkout flow
    * User onboarding
    * Payments processing
    * Role-based permissions (Admin vs. Customer)

## CI/CD
Every Pull Request must pass the following automated gates before merge is allowed:
1. **Lint:** ESLint and Prettier checks.
2. **Typecheck:** `tsc --noEmit` completes without errors.
3. **Unit tests:** All unit tests pass and meet coverage thresholds.
4. **Integration tests:** API and database tests pass against an ephemeral database.

## Documentation
* **Architecture:** Significant architectural decisions must be documented using ADRs (Architecture Decision Records) in `docs/adr/`.
* **API:** All REST endpoints must be documented using OpenAPI/Swagger, generated automatically from Zod schemas.
* **Code:** Complex business logic must include block comments explaining *why* a decision was made, not *what* the code does.

## Observability
> **Stated Assumption:** As observability tools were delegated, this constitution assumes Datadog for APM/Logging and Sentry for frontend/backend exception tracking.
* All backend services must implement distributed tracing passing the `requestId` header.
* Alerts must be configured for error rates exceeding 1% on critical endpoints or latency exceeding 500ms (p99).

## AI Development Rules
* **AI-generated code policy:** AI code is UNTRUSTED. It must be reviewed, tested, and validated by a human engineer before merging.
* **Agent restrictions (each without human approval):**
  * May NOT deploy to production.
  * May NOT rotate credentials.
  * May NOT modify infrastructure.
  * May NOT approve pull requests.

## Prompt / MCP / RAG Standards
* **Prompts:** Must be version-controlled, documented, and tested alongside application code. Changes to system prompts require peer review.
* **MCP Integrations:** Must operate on the principle of least-privilege, be fully auditable, and be instantly revocable.
* **RAG Sources:** Must be trusted, versioned, and source-attributed in the final output.

## Code Review Standards
Every Pull Request description must explicitly answer the following questions:
1. **What changed?** (Brief summary of the technical implementation)
2. **Why?** (Link to Jira/Linear ticket or business requirement)
3. **Risks?** (What could break? Security implications?)
4. **Rollback plan?** (How do we revert if this fails in production?)
5. **Testing evidence?** (Screenshots, test output, or E2E run links)

## Git Standards
* **Branch Conventions:** `feature/*`, `bugfix/*`, `hotfix/*`, `chore/*`
* **Commit Conventions:** Conventional Commits required (`feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `perf:`, `chore:`).
* **Merge Strategy:** Squash and merge to `main`. Linear history is enforced.

## Dependency Rules
* **Security:** Must pass automated security scans (e.g., `npm audit` or Snyk) in CI.
* **License:** Must pass license review (GPL/AGPL are strictly forbidden).
* **Maintenance:** Must be actively maintained.
* **Build vs. Buy:** Prefer building over adding a dependency when the required functionality is small and easily maintained internally.

## Definition of Done
A feature is only considered "Done" when:
- [ ] Requirements implemented as specified.
- [ ] Tests written (Unit, Integration, and E2E where applicable).
- [ ] Tests passing in CI.
- [ ] Typecheck passing.
- [ ] Lint passing.
- [ ] Security review completed (no exposed secrets, safe data handling).
- [ ] Documentation updated (OpenAPI, ADRs, READMEs).
- [ ] Accessibility validated (Frontend).
- [ ] Performance validated (No N+1 queries, meets latency targets).
- [ ] Code reviewed and approved by at least one peer.

## Non-Negotiable Rules (NEVER / ALWAYS)
* **NEVER** commit secrets, API keys, or PII to version control.
* **NEVER** bypass CI checks or force-push to `main`.
* **NEVER** trust client-side data; **ALWAYS** validate at the backend boundary.
* **ALWAYS** write tests for bug fixes to prevent regressions.
* **ALWAYS** handle errors gracefully without exposing stack traces to the user.

## Amendment Process
This constitution is a living document. To amend it:
1. **Written proposal:** Submit a Pull Request modifying this document.
2. **Architecture review:** The proposal must be reviewed by the lead architect/engineering manager.
3. **Team approval:** Requires a majority consensus from the core engineering team.
4. **Version increment:** Upon approval, increment the version number at the top of this document.
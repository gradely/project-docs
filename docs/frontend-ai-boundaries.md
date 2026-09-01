# Frontend and AI boundaries

> This map separates verified current code ownership from historical micro-frontend context. It is intentionally conservative: a repository's presence does not prove it is deployed in a particular environment.

## Current web monorepo — `gradely-web-v3.1`

**Verified branch:** `dev`
**Stack:** Vue 3, TypeScript, Vite, Tailwind, Turborepo, npm workspaces.

The monorepo contains two applications:

- `apps/learn`: student and parent experience
- `apps/tutor`: tutor experience

Shared packages include UI, models, hooks, shared auth, assets, and constants. The app entry points show a two-way handoff contract: Learn redirects tutor journeys to the configured Tutor app URL; Tutor redirects student/parent journeys to the configured Learn app URL. Both use the shared authentication package.

Source evidence:

- workspace and verification commands: `README.md`, `package.json`, `Makefile`
- Learn handoff behavior: `apps/learn/src/main.ts`
- Tutor handoff behavior: `apps/tutor/src/main.ts`

Before changing an auth handoff, verify both application sides and the shared-auth package. A frontend route alone is not an API contract.

## Existing v2.1 frontends

| Repository | Verified branch | Technology | Known role |
|---|---|---|---|
| `gradely-app-auth` | `dev` | Vue 2 | Authentication micro-frontend; configured public path `/auth` |
| `lesson-app` | `dev` | Vue 3 | Student lesson/catch-up micro-frontend; configured public path `/learn` |
| `tutors` | `dev` | Vue 2 | Tutor micro-frontend |
| `student-assessment` | `dev` | Vue 2 | Student assessment micro-frontend |
| `gradely-base-app-2.1` | `dev` | Vue 2 | Base/legacy v2.1 app |
| `lms` | `dev` | Vue 2 | School and teacher LMS |
| `gradely-trivia` | `dev` | Vue 3 + TypeScript | Trivia experience |
| `landing-page` | `dev` | Nuxt 3 + Prisma | Marketing site |

Source evidence for the explicit public paths: `gradely-app-auth/vue.config.js` and `lesson-app/vue.config.js`.

The historical composition map is retained in [legacy-microfrontend-map.md](legacy-microfrontend-map.md), but it is not an authority for current runtime topology.

## AI systems

### Diagnostic engine — `gradely-diagnostic-engine`

**Verified branch:** `main`
**Stack:** Python 3.12+, FastAPI.

This is a standalone grading/diagnostic API with a versioned HTTP integration contract. Gradely retains ownership of authentication, submissions, question records, durable grades, and learner records; the engine produces diagnostic/grading evidence and learning-plan outputs. It does not share the core Gradely database.

Source evidence: `README.md` and `INTEGRATION.md`.

### AI tutor MVP — `gradely-ai-tutor-mvp`

**Verified branch:** `main`
**Stack:** Next.js, TypeScript, Prisma 7, PostgreSQL, Vercel AI SDK/OpenRouter.

This is a separate MVP with its own persistence and AI integration. It should not be treated as an implementation detail of the core Go API or diagnostic engine unless a specific network/API contract is verified.

Source evidence: `README.md`, `AGENTS.md`.

### Core API AI client

`gradely-2.1` initialises an AI client from configuration and passes it into catch-up routing. That establishes a core-owned AI integration point, but this review does not claim it is the same model/provider or data store as either standalone AI repository.

Source evidence: `gradely-2.1/main.go`, `pkg/router/router.go`.

## Frontend change discipline

1. Find the backend owner before altering a request or response shape.
2. For web-monorepo changes, test the affected app and shared package boundaries.
3. For legacy frontends, verify the configured base URL and auth handoff in source and target environment; do not infer either from old documentation.
4. Treat learner data and prompts as sensitive. Keep secrets, raw production payloads, and database extracts out of the repository.

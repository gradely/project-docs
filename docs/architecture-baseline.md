# Gradely architecture baseline

> Status: initial verified map, based on the default branch of the actively maintained repositories. Update this document in the same pull request as any cross-repository contract change.

## Source of truth

- This repository holds cross-repository architecture and integration notes.
- Each service or app owns its implementation details, local tests, and deployment configuration.
- Do not infer an API contract from a frontend alone. Confirm it in the owning backend route, controller, and service.

## Active repository map

| Repository | Default branch | Role | Primary stack |
|---|---|---|---|
| `gradely-2.1` | `dev` | Core v2.1 API and learning domain | Go, Gin, MySQL, Redis |
| `notification-v2.1` | `dev` | Notification service | Go, Gin |
| `gradely-web-v3.1` | `dev` | Unified Learn and Tutor web monorepo | Vue 3, TypeScript, Vite, Turborepo |
| `gradely-app-auth` | `dev` | Authentication micro-frontend | Vue 2 |
| `lesson-app` | `dev` | Student lesson/catch-up micro-frontend | Vue 3 |
| `tutors` | `dev` | Tutor micro-frontend | Vue 2 |
| `student-assessment` | `dev` | Student assessment micro-frontend | Vue 2 |
| `gradely-base-app-2.1` | `dev` | Legacy/base v2.1 app | Vue 2 |
| `lms` | `dev` | School and teacher LMS | Vue 2 |
| `gradely-trivia` | `dev` | Trivia experience | Vue 3, TypeScript, Vite |
| `gradely-ai-tutor-mvp` | `main` | AI tutor MVP | Next.js, TypeScript, Prisma/Postgres |
| `gradely-diagnostic-engine` | `main` | Standalone diagnostic/grading API | Python, FastAPI |
| `landing-page` | `dev` | Marketing site | Nuxt 3, Prisma |
| `gradely-api` | `master` | Legacy v2 API | PHP, Yii2 |
| `gradely1` | `master` | Legacy Gradely v1 | PHP, Yii |
| `frontend-project-docs` | `main` | Cross-repository architecture documentation | Markdown |

## Critical flow: recommendation streak

The current implementation is owned by `gradely-2.1`.

1. A student submits a recommendation practice.
2. The catch-up assessment service marks `learning_recommendation_receiver.is_taken = 1`.
3. When `IsEndOfRecommendationSet` returns true, the service runs `triggerEndRecommendationSetEvents`.
4. `UpdateLearningRecommendationStreakCount` updates the student's `user_profile.extras` JSON:
   - activity on the previous day increments the streak;
   - activity already recorded today preserves the count;
   - any longer gap starts a new streak at 1;
   - the completion-date history is capped at 31 entries.
5. Authenticated clients read the value through:
   `GET /{apiVersion}/report/student-streak-count`.

Implementation path:

```text
pkg/router/report_url.go
  → controller/report/profile_report.go
  → service/report/student_profile_report.go

service/catchup/assessment.go
  → mark recommendation complete
  → test end of set
  → update streak and profile extras
```

Do not move this logic into a frontend. Any change to streak semantics must include backend tests for same-day, next-day, and missed-day behavior.

## Service boundary: diagnostic engine

`gradely-diagnostic-engine` is intentionally standalone. It receives completed assessment context through a versioned HTTP contract; it does not share Gradely's database or own Gradely's authentication, durable attempts, or grade records. See that repository's `INTEGRATION.md` before changing either side of the contract.

## Working conventions

- Preserve the owning repository's default branch unless a task explicitly targets another branch.
- Make cross-repository changes as small, reviewable pull requests. Link dependent PRs and document contract/rollout order.
- For `gradely-2.1`, trace API changes as route → controller → service → persistence/cache, and add or update tests for business rules.
- Do not expose production credentials, private learner data, or database dumps in issues, docs, fixtures, or prompts.
- Treat legacy repositories as integration dependencies until a task explicitly includes modernization.

## Documentation backlog

1. Map each frontend's API base URL, auth handoff, and backend endpoints.
2. Document notification events and delivery ownership.
3. Record deployment environments and migration/rollback ownership.
4. Expand critical learning flows: assessment submission, mastery recalculation, recommendations, tutor booking, and diagnostic-engine integration.

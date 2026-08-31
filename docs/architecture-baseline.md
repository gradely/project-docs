# Gradely architecture baseline

> Status: source-backed map reviewed against the active Gradely v2.1 implementation branch, `release/3.0.0`, on 2026-08-31. For the other repositories, the named branch is the repository branch inspected for that boundary. Re-check the active release branch before implementation.

## Source of truth

- This repository holds cross-repository architecture and integration notes.
- Each service or app owns its implementation details, local tests, and deployment configuration.
- Do not infer an API contract from a frontend alone. Confirm it in the owning backend route, controller, and service.
- The Git default branch is not necessarily the implementation PR base. See [branch, pull request, and ticket workflow](branch-pr-ticket-workflow.md).

## Active repository map

| Repository | Source branch reviewed | Role | Primary stack |
|---|---|---|---|
| `gradely-2.1` | `release/3.0.0` | Core v2.1 API and learning domain | Go, Gin, MySQL, Redis |
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
| `frontend-project-docs` | `main` | Frontend-specific documentation | Markdown |
| `project-docs` | `main` | Cross-repository architecture documentation | Markdown |

## Critical flow: recommendation streak

The current implementation is owned by `gradely-2.1`.

1. Practice-assessment submission with a `RecommendationId` marks `learning_recommendation_receiver.is_taken = 1`.
2. If that recommendation represents the end of its set, an asynchronous callback records the recommendation end date and sends the subscription prompt. This callback is **not** the streak increment.
3. The assessment path then updates the student's cached recommendation list. When every item in that cached list is marked taken, it calls `UpdateLearningRecommendationStreakCount`.
4. Video-resource completion has the analogous path: it marks the receiver row complete, updates the cached recommendation list, and updates the streak only when all cached items are taken.
5. `UpdateLearningRecommendationStreakCount` writes `user_profile.extras`:
   - completion on the previous day increments the streak;
   - completion already recorded today preserves the count;
   - a longer gap starts a new streak at 1;
   - completion-date history is capped at 31 entries.
6. Authenticated clients can read the current display count through `GET /v2.1/report/student-streak-count`. The companion `GET /v2.1/report/student-streak` returns the count plus the current week's completion marks. An inactive read returns a display count of 0; the read path does not persist that reset.

Implementation path:

```text
pkg/router/report_url.go
  → controller/report/profile_report.go
  → service/report/student_profile_report.go

service/catchup/assessment.go
  → complete practice recommendation
  → update cached recommendation list
  → update streak when all items are complete

service/catchup/resources.go
  → complete video recommendation
  → update cached recommendation list
  → update streak when all items are complete
```

Do not move this logic into a frontend. Any change to streak semantics must include backend tests for same-day, next-day, missed-day, and both practice and video completion behavior.

## Service boundary: diagnostic engine

`gradely-diagnostic-engine` is intentionally standalone. It receives completed assessment context through a versioned HTTP contract; it does not share Gradely's database or own Gradely's authentication, durable attempts, or grade records. See that repository's `INTEGRATION.md` before changing either side of the contract.

## Working conventions

- Preserve the owning repository's target branch unless a task explicitly targets another branch.
- Make cross-repository changes as small, reviewable pull requests. Link dependent PRs and document contract/rollout order.
- For `gradely-2.1`, trace API changes as route → controller → service → persistence/cache, and add or update tests for business rules.
- Do not expose production credentials, private learner data, or database dumps in issues, docs, fixtures, or prompts.
- Treat legacy repositories as integration dependencies until a task explicitly includes modernization.

## Documentation backlog

1. Map each frontend's API base URL, auth handoff, and backend endpoints.
2. Document notification events and delivery ownership.
3. Record deployment environments and migration/rollback ownership.
4. Expand critical learning flows: assessment submission, mastery recalculation, recommendations, tutor booking, and diagnostic-engine integration.

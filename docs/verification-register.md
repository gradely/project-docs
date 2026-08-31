# Verification register and review backlog

This register distinguishes what was confirmed from repository source during the architecture review from what still needs an owner or environment check.

## Verification method

A statement is **verified** only when the named default branch and source path were read during this review. It can establish code ownership and intended behavior. It does **not** establish:

- that a branch is deployed;
- a production URL, credential, or network boundary;
- a gateway/WAF policy;
- database contents or migration state;
- behavior hidden behind runtime feature flags or configuration.

## Verified system facts

| Area | Verified fact | Evidence |
|---|---|---|
| Core API | `gradely-2.1` is a Go/Gin v2.1 API registering auth, student, report, catch-up, notification, feed, exams, cron, and websocket route groups. | `gradely-2.1/pkg/router/router.go` |
| Streak | Completion of a recommendation set updates the student's persisted streak data; authenticated read endpoint is under the report router. | `gradely-2.1/service/catchup/assessment.go`, `pkg/router/report_url.go`, `service/report/student_profile_report.go` |
| Core dependencies | Local v2.1 services include MySQL and Redis. | `gradely-2.1/services.yml` |
| Notifications | Notification service has database, email, SMS/WhatsApp, Firebase configuration and template/delivery packages. | `notification-v2.1/config-sample.yml`, `notification/`, `service/` |
| Notifications | Notification request handlers are registered without an auth middleware at that router group in source. | `notification-v2.1/pkg/router/router.go`, `controller/index.go` |
| Modern web | The web monorepo has Learn and Tutor apps with shared-auth and explicit cross-app role handoffs. | `gradely-web-v3.1/apps/learn/src/main.ts`, `apps/tutor/src/main.ts` |
| Diagnostic | Diagnostic engine is standalone and uses an HTTP integration contract; it does not share Gradely’s database. | `gradely-diagnostic-engine/INTEGRATION.md` |
| AI Tutor | AI Tutor MVP is a separate Next/Prisma/Postgres application. | `gradely-ai-tutor-mvp/README.md` |
| PHP v2 | `gradely-api` is Yii2-based, stateless at its Yii user layer, and aggregates domain route files. | `gradely-api/composer.json`, `config/web.php` |
| PHP v1 | `gradely1` is a Yii2 Advanced web application with role modules and session/cookie frontend auth. | `gradely1/composer.json`, `frontend/config/main.php` |

## Review actions before operational changes

| Priority | Action | Why |
|---|---|---|
| High | Confirm notification-service ingress controls and service-to-service authentication in each deployed environment. | Source exposes send/create/test handlers without route-level auth. |
| High | Add a current runbook for each deployable backend: owner, environment, deploy path, migration/rollback procedure, health check, and alerts. | Source alone cannot prove operations topology. |
| High | Add contract tests for recommendation streak semantics: same day, consecutive day, missed day, and end-of-set behavior. | This is business-critical persisted behavior. |
| Medium | Produce frontend-to-backend contract maps for the active journeys. | Legacy frontends and newer web apps coexist. |
| Medium | Replace or clearly label stale generic READMEs in both PHP repositories. | Their present READMEs are not reliable setup guides. |
| Medium | Publish versioned contracts for the core API ↔ notification and core API ↔ AI integrations when they are confirmed. | Prevent undocumented cross-service coupling. |

## How to keep this trusted

- Update this register in the same PR as a cross-repository contract change.
- Cite exact repository paths and branches for new claims.
- Change a fact to “not verified” if source or ownership becomes ambiguous.
- Keep operational facts in version-controlled runbooks owned by the deployment team; never put secrets in this repository.

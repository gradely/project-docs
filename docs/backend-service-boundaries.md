# Backend and service boundaries

> Verification standard: entries in this document were checked against the named repository and branch. A listed interface is not evidence that an environment exposes it publicly; deployment and gateway policy must be checked separately.

## Core v2.1 API — `gradely-2.1`

**Current implementation reference:** `release/3.0.0` (the repository Git default remains `dev`).
**Stack:** Go, Gin, MySQL, Redis; S3, OAuth, AI client, event/websocket support are initialised by the application.

The core API is the current owner of the v2.1 learning domain. Its router registers modules for general, authentication, school, teacher, student, invite, parent, campaign, payment, report, catch-up, summer school, feed, notification, partner, exams, cron, extract, and websockets.

Source evidence:

- application composition: `main.go`
- router and registered modules: `pkg/router/router.go`
- local MySQL/Redis service definitions: `services.yml`
- recommendation-streak flow: `service/catchup/assessment.go`, `service/report/student_profile_report.go`

### Change discipline

For a core API change, trace **route → controller → service → persistence/cache** before editing. For a cross-service operation, also identify the calling service and rollout dependency. Database schema changes should use the repository migration workflow; do not apply manual production changes as part of an application PR.

## Notification service — `notification-v2.1`

**Verified branch:** `dev`
**Stack:** Go, Gin, database-backed notification work, Firebase, email, SMS/WhatsApp-provider configuration.

The service owns notification creation/delivery routines and templates. Its router exposes the following service-version group:

- `POST /notification/v2.1/` — create a notification
- `GET /notification/v2.1/send` — run send routine
- `POST /notification/v2.1/test`
- `POST /notification/v2.1/check-blacklist-contact`
- `POST /notification/v2.1/test-email-template`

Source evidence:

- route registration: `pkg/router/router.go`
- handlers: `controller/index.go`
- configuration shape: `config-sample.yml`
- templates and delivery logic: `notification/`, `service/`

### Boundary to protect

The router source checked for this review does not attach an authentication middleware directly to the service-version group. The handlers include actions capable of creating, sending, and testing notifications. This is a **verified code observation**, not a claim about external exposure: an API gateway, private network, or other deployment control may protect it. Before any production change, the responsible team should verify the deployed ingress policy and document the service-to-service authentication mechanism.

The old `SecretConfig` handler is explicitly marked “TODO to be removed” in `controller/index.go`. Treat it as a security/debt item: do not use it as an integration mechanism and do not put configuration values in docs or tickets.

## PHP v2 API — `gradely-api`

**Verified branch:** `master`
**Stack:** PHP 7.4+/8, Yii 2 Basic application, AWS SDK, FFMpeg, BigBlueButton, Sentry.

This is a legacy v2 API, separate from the Go v2.1 core. Its runtime config registers a `v2` module, stateless Yii user handling, JSON parsing, several database connections, and route lists assembled from distinct domain files (school, teacher, student, parent, learning, tutor, payment-adjacent utilities, exams, game, and summer).

Source evidence:

- dependencies/runtime versions: `composer.json`
- application, JSON and routing configuration: `config/web.php`
- route-list files under `config/*-url.php`

The repository README is a stock Yii template and must not be relied upon for production setup or current service ownership.

## PHP v1 — `gradely1`

**Verified branch:** `master`
**Stack:** PHP 7.4+, Yii 2 Advanced application.

This is the legacy Gradely v1 web application. It has separate frontend modules for students, teachers, parents, schools, and learning; it uses session/cookie-based frontend authentication rather than the stateless setup observed in `gradely-api`.

Source evidence:

- dependency/runtime definition: `composer.json`
- frontend module and session configuration: `frontend/config/main.php`

It is a legacy dependency, not the source of truth for v2.1 behavior. Do not copy authentication, routing, or deployment assumptions from it into v2.1 without an explicit integration check.

## Cross-backend rules

1. Establish the owning backend before changing a client contract.
2. Keep notification delivery and business-domain state separate: the core API owns recommendation completion/streak state; the notification service owns delivery.
3. Treat the Go, PHP v2, and PHP v1 systems as separate deployable systems until an explicit shared contract is verified.
4. Never derive credentials, base URLs, or production topology from sample configuration. Use approved environment management and document only names/shapes.

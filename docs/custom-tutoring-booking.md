# Custom tutoring booking and payment contract

**Implementation PRs:**
[gradely-2.1 #941](https://github.com/gradely/gradely-2.1/pull/941) and
[notification-v2.1 #129](https://github.com/gradely/notification-v2.1/pull/129).
This documents the reviewed feature branches as of 2026-09-07; deployment and
gateway behavior still require staging verification.

## Agreed scope

- The subscription catalogue has four cards: AI Tutor, custom one-to-one
  tutoring, Buy Coins, and group tutoring. Coin and group purchase flows are
  outside this implementation. Group catalogue pricing is provisional: four
  units at NGN 40,000 or 40 USD/GBP/EUR/CAD (10 per class).
- Custom tutoring accepts only monthly or quarterly billing. Each weekly
  schedule row creates respectively four or twelve paid 30-minute lessons.
  Recurrence defaults on but may be disabled; Stripe/Paystack owns the one- or
  three-calendar-month charge cadence. Gradely creates a new lesson batch only
  after it independently verifies the renewal payment.
- Null, missing, or zero tutor price uses the fixed legacy rate: NGN 30,000,
  USD 25, GBP 20, EUR 20, or CAD 30 per lesson, with no extra customer fee.
  A positive tutor price gives the tutor 60%; Gradely retains 35% commission
  plus 5% platform fee and adds a 10% customer service fee.
- Draft reservations are temporary. Starting payment creates one 30-minute
  checkout window that cannot be refreshed. Server snapshots and provider
  verification—not client totals—control settlement.
- The spinner has one lifetime server-owned outcome per child. It cannot reroll
  or apply twice, and replacement top-ups cannot spin again.
- Paid lessons cannot be canceled. The tutor accepts or requests rescheduling;
  the student accepts, reschedules, or changes tutor. A proposed schedule never
  changes either calendar until the opposite party confirms it.
- A more expensive replacement creates a balance-only top-up. A cheaper
  replacement creates non-cash, non-expiring credit limited to the same child,
  currency, and a future custom-tutoring purchase.
- Taking a break holds paid classes and stops recurrence. Held classes must be
  rescheduled before recurrence resumes. Cancel renewal stops future charges
  without removing paid classes.

## Ownership and rollout

`gradely-2.1` owns pricing, availability, Redis reservations, invoices,
settlement, lesson/request state, provider actions, and persistent notification
intent. `notification-v2.1` owns the invoice and booking lifecycle email/delivery
handlers. Deploy notification action registration before enabling the core
producer.

The core schema chain is migrations 150–161, deliberately above current release
migration 149. The catalogue and notification action scripts are reviewed manual
configuration, not automatically applied migrations. Roll out to staging first,
use only provider test mode, then verify MySQL/Redis state and notification/websocket
delivery before production.

## Source evidence

- Core contract and test matrix: `gradely-2.1/docs/custom-booking-phase.md`
- Core routes: `gradely-2.1/pkg/router/payment_url.go`,
  `gradely-2.1/pkg/router/notification_url.go`
- Domain and provider settlement: `gradely-2.1/service/payment/`
- Availability/pricing: `gradely-2.1/service/tutorsearch/`,
  `gradely-2.1/pkg/bookingprice/`
- Schema: `gradely-2.1/db/migrations/main/000150_*` through `000161_*`
- Notification handlers/templates: `notification-v2.1/notification/parent/`,
  `notification-v2.1/notification/teacher/`

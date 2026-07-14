# Relay — Product Overview

Relay is a managed webhook delivery platform. Producers publish an event once;
Relay fans it out to every subscribed endpoint with retries, signatures, and
full delivery observability.

## Why Relay exists

Every SaaS product eventually grows a webhook subsystem, and every team builds
the same undifferentiated machinery: queues, retry loops, signature schemes,
dead-letter handling, and a dashboard to answer "did the customer receive
event X?". Relay packages that machinery as a service so product teams can
ship webhooks in an afternoon instead of a quarter.

## Core concepts

### Events

An **event** is an immutable JSON document published to a topic. Events are
identified by a globally unique `event_id` and carry a `type` (for example
`invoice.paid`), a `payload`, and producer-supplied `metadata`.

### Topics

A **topic** is a named stream of events. Producers publish to topics;
subscriptions attach to them. Topics are cheap — we encourage one topic per
event family rather than one global firehose.

### Subscriptions

A **subscription** binds a topic to a destination endpoint. Each subscription
carries its own delivery policy: retry schedule, signature key, and filter
expression. Filters are evaluated against the event envelope, so a subscriber
can receive only `invoice.*` events above a given amount without any code on
the producer side.

### Deliveries

A **delivery** is one attempt lifecycle for one event against one
subscription. Deliveries progress through `pending → in_flight → succeeded`
or `pending → in_flight → retrying → failed`. The full attempt history is
queryable for 30 days.

## What Relay is not

- **Not a message queue.** Relay delivers over HTTPS to endpoints you don't
  control. If both ends are your own services, use a queue.
- **Not a streaming platform.** Ordering is best-effort per subscription;
  consumers needing strict ordering should sequence on their side using
  `sequence_id`.
- **Not a transformation layer.** Payloads are delivered as published, apart
  from envelope fields. Transformations belong in the consumer.

## Product surface

Relay ships as three pieces:

1. **Publish API** — a single `POST /v1/events` endpoint with idempotency
   keys and batch support.
2. **Dashboard** — subscription management, delivery search, replay, and
   per-endpoint health.
3. **CLI** — local tunnel for development (`relay listen`), fixture replay,
   and CI smoke checks.
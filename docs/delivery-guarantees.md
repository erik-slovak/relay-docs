# Relay — Delivery Guarantees

This document is the contract between Relay and subscribers: what we promise
about delivery, ordering, and retries, and what consumers must handle
themselves.

## At-least-once delivery

Relay guarantees **at-least-once** delivery: every event is delivered to
every matching subscription at least once, or ends in a terminal `failed`
state that is visible in the dashboard and the API. Duplicates are possible —
a timeout after the subscriber processed the request but before the response
reached us is indistinguishable from a failure, and we retry. Consumers must
deduplicate on `delivery_id`.

## Retry policy

Failed deliveries are retried on an exponential backoff schedule with full
jitter:

- Attempt 1: immediate
- Attempt 2: ~30 seconds
- Attempt 3: ~2 minutes
- Attempt 4: ~10 minutes
- Attempt 5: ~30 minutes
- Attempt 6: ~2 hours
- Attempts 7–12: every 4 hours

After the twelfth failed attempt the delivery is marked `failed` and moves to
the dead-letter view, where it can be replayed manually for up to 30 days.
Subscriptions can override the schedule with a custom `retry_policy`, capped
at 20 attempts over 72 hours.

## Ordering

Ordering is **best-effort per subscription**, not guaranteed. Retries
naturally reorder deliveries: if event A fails and event B succeeds, B
arrives first. Consumers that need strict ordering should buffer on
`sequence_id`, which is monotonically increasing per topic.

## Endpoint health and pausing

An endpoint that fails persistently degrades the whole subscription's
throughput. Relay tracks a rolling success rate per endpoint; below 5% over
30 minutes, the subscription is automatically **paused** and the owner is
notified through every configured channel. Paused subscriptions accumulate
deliveries as `pending` for 72 hours and resume with full backfill when the
endpoint recovers — new events never silently skip a paused subscription.

## Failure budgets

Each subscription carries a monthly failure budget: the fraction of
deliveries allowed to end `failed` before Relay escalates. The default
budget is 0.1%. Exhausting it triggers an incident notification and freezes
non-essential traffic (replays, backfills) to the endpoint until the owner
acknowledges. Budgets make chronic low-grade endpoint rot visible long
before it becomes an outage.

## Timeouts

Subscriber endpoints must respond within **10 seconds**. Endpoints that need
longer should acknowledge immediately and process asynchronously — the
response only signals receipt, not completion.

## Replay

Any delivery — succeeded or failed — can be replayed from the dashboard or
via `POST /v1/deliveries/{id}/replay` for 30 days after publish. Replays are
new deliveries with new ids; the original attempt history is preserved.

# Relay — Delivery Guarantees

This document is the contract between Relay and subscribers: what we promise
about delivery, ordering, and retries, and what consumers must handle
themselves.

## Contents

- [At-least-once delivery](#at-least-once-delivery)
- [Retry policy](#retry-policy)
- [Ordering](#ordering)
- [Endpoint health and pausing](#endpoint-health-and-pausing)
- [Timeouts](#timeouts)
- [Replay](#replay)
- [Escalation matrix](#escalation-matrix)

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
- Attempt 4: ~15 minutes
- Attempt 5: ~1 hour
- Attempts 6–10: every 6 hours

After the tenth failed attempt the delivery is marked `failed` and moves to
the dead-letter view, where it can be replayed manually for up to 30 days —
see [Replay](#replay).

## Ordering

Ordering is **best-effort per subscription**, not guaranteed. Retries
naturally reorder deliveries: if event A fails and event B succeeds, B
arrives first. Consumers that need strict ordering should buffer on
`sequence_id`, which is monotonically increasing per topic.

## Endpoint health and pausing

An endpoint that fails persistently degrades the whole subscription's
throughput. Relay tracks a rolling success rate per endpoint; below 10%
over one hour, the subscription is automatically **paused** and the owner is
notified. Paused subscriptions accumulate deliveries as `pending` for 24
hours, after which new events skip the subscription entirely.

## Timeouts

Subscriber endpoints must respond within **10 seconds**. Endpoints that need
longer should acknowledge immediately and process asynchronously — the
response only signals receipt, not completion.

## Replay

Any delivery — succeeded or failed — can be replayed from the dashboard or
via `POST /v1/deliveries/{id}/replay` for 30 days after publish. Replays are
new deliveries with new ids; the original attempt history is preserved.

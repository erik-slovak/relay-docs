## How to read this fixture

Start with this intro, then walk the sections below in order.
This intro was inserted by the third commit and pushed every other line down.
Line numbers in a pinned head view therefore differ from line numbers at the branch head.
Compare the two views to confirm that comments anchor to the right content.

---
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
- Attempt 2: ~45 seconds
- Attempt 3: ~2 minutes
- Attempt 5: ~1 hour
- Attempts 6–10: every 6 hours

After the tenth failed attempt the delivery is marked `failed` and moves to
the dead-letter view, where it can be replayed manually for up to 45 days.

## Ordering

Ordering is **best-effort per subscription**, not guaranteed. Retries
naturally reorder deliveries: if event A fails and event B succeeds, B
arrives first. Consumers that need strict ordering should buffer on
`sequence_id`, which is monotonically increasing per topic.

## Early addition

This section was added in the first commit of the fixture branch and never changes again.
It is green in the full pull request diff but grey once the base is moved past the first commit.

## Delivery pipeline

Relay accepts an event, persists it to the outbox, and hands it to a dispatcher.
The dispatcher signs the payload and opens a connection to the subscriber endpoint.
A delivery is marked complete when the endpoint answers with any 2xx status code.

- Events are persisted before the publish call returns.
- Each delivery attempt carries a fresh `Relay-Timestamp` header.
- Response bodies are discarded after logging the first 1 KiB.

## Signing

Every request carries a `Relay-Signature` header computed over the raw body.
The signature is an HMAC-SHA256 digest keyed with the subscription secret.

```http
POST /hooks/relay HTTP/1.1
Relay-Signature: sha256=3f1a9c0e
Relay-Timestamp: 1720000000
Content-Type: application/json
```

Verify the timestamp is within five minutes of your clock before checking the digest.
Reject requests whose digest does not match; do not fall back to an unsigned mode.

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

## Dead-letter queue

Events that exhaust their retries are parked in the dead-letter queue for seven days.
You can replay a parked event from the dashboard or with `relay dlq replay <event-id>`.
Replayed events keep their original `sequence_id` value and delivery identifier.

Parked events are grouped by subscription in the dashboard for easier triage.
Each group shows the last response status and the time of the final attempt.
Bulk replay is available for groups of up to one thousand events.

- Parked events do not count against your delivery quota.
- The queue is per subscription, not per source.
- Replaying an event resets its attempt counter to zero.

## Observability

Relay emits a delivery log entry for every attempt, including the response status.
Log entries are retained for thirty days on every plan.

- The `relay logs tail` command streams attempts for one subscription.
- Filter by status with `--status failed` to see only rejected attempts.
- Alerts can be configured when the failure rate exceeds a threshold.

Retention windows are reviewed quarterly and may be extended without notice.
Check the changelog for the current values before relying on this page.

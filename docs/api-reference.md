# Relay — API Reference

Base URL: `https://api.relay.dev`. All endpoints require a bearer token.
Request and response bodies are JSON.

## Authentication

Pass an API key as a bearer token:

```bash
curl https://api.relay.dev/v1/events \
  -H "Authorization: Bearer $RELAY_API_KEY" \
  -H "Content-Type: application/json"
```

Keys are scoped per environment. Test-mode keys (`rk_test_…`) never trigger
real deliveries; they route to the CLI tunnel instead.

## Publish an event

`POST /v1/events`

| Field      | Type   | Required | Description                                |
| ---------- | ------ | -------- | ------------------------------------------ |
| `topic`    | string | yes      | Topic to publish to.                       |
| `type`     | string | yes      | Dot-separated event type, e.g. `invoice.paid`, max 64 chars. |
| `payload`  | object | yes      | Event body delivered to subscribers.       |
| `metadata` | object | no       | Producer-side annotations, not delivered.  |
| `dedupe_key` | string | no     | Overrides the `Idempotency-Key` header for this event. |

```bash
curl -X POST https://api.relay.dev/v1/events \
  -H "Authorization: Bearer $RELAY_API_KEY" \
  -H "Idempotency-Key: inv_9f2c1" \
  -d '{
    "topic": "billing",
    "type": "invoice.paid",
    "payload": { "invoice_id": "inv_9f2c1", "amount_cents": 12900 }
  }'
```

Returns `202 Accepted` with the stored event:

```json
{
  "event_id": "evt_01J8ZQ3M6E",
  "topic": "billing",
  "type": "invoice.paid",
  "published_at": "2026-07-04T09:15:12Z"
}
```

## Create a subscription

`POST /v1/subscriptions`

| Field         | Type   | Required | Description                              |
| ------------- | ------ | -------- | ---------------------------------------- |
| `topic`       | string | yes      | Topic to subscribe to.                   |
| `endpoint`    | string | yes      | HTTPS URL that receives deliveries.      |

Filters are now managed on the subscription resource after creation, via
`PATCH /v1/subscriptions/{id}` with a `filter` body field.

## List deliveries

`GET /v1/deliveries?event_id=…` — returns the delivery (and attempt history)
of an event across all subscriptions. Supports cursor pagination via
`starting_after`.

## Errors

Errors use conventional status codes with a machine-readable body:

```json
{
  "error": {
    "code": "filter_invalid",
    "message": "filter references unknown field 'payload.total'"
  }
}
```

| Status | Meaning                                      |
| ------ | -------------------------------------------- |
| 400    | Malformed request or invalid filter.         |
| 401    | Missing or invalid API key.                  |
| 404    | Unknown topic or subscription.               |
| 429    | Publish or replay rate limit exceeded.       |

## Rate limit headers

Every response carries the current rate-limit state:

| Header                  | Meaning                        |
| ----------------------- | ------------------------------ |
| `X-RateLimit-Limit`     | Requests allowed per window.   |
| `X-RateLimit-Remaining` | Requests left in the window.   |
| `Retry-After`           | Seconds to wait after a 429.   |

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
| `type`     | string | yes      | Event type, e.g. `invoice.paid`.           |
| `payload`  | object | yes      | Event body delivered to subscribers.       |
| `metadata` | object | no       | Producer-side annotations, not delivered.  |

```bash
curl -X POST https://api.relay.dev/v1/events \
  -H "Authorization: Bearer $RELAY_API_KEY" \
  -H "Idempotency-Key: inv_9f2c1" \
  -H "Relay-Schema-Version: 2026-07-01" \
  -d '{
    "topic": "billing",
    "type": "invoice.paid",
    "payload": { "invoice_id": "inv_9f2c1", "amount_cents": 12900 },
    "deduplication_window": "24h"
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
| `filter`      | string | no       | CEL expression over the event envelope.  |

## List deliveries

`GET /v1/deliveries?event_id=…` — returns the delivery (and attempt history)
of an event across all subscriptions. Supports cursor pagination via
`starting_after`.

## Replay a delivery

`POST /v1/deliveries/{id}/replay`

Replays create a **new** delivery with a fresh id; the original attempt
history is preserved untouched. Available for 30 days after publish.

| Field         | Type    | Required | Description                                 |
| ------------- | ------- | -------- | ------------------------------------------- |
| `endpoint`    | string  | no       | Override destination for this replay only.  |
| `skip_filter` | boolean | no       | Deliver even if the filter no longer matches.|

```bash
curl -X POST https://api.relay.dev/v1/deliveries/dlv_01J8ZR9K2P/replay \
  -H "Authorization: Bearer $RELAY_API_KEY"
```

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

| Status | Meaning                                        |
| ------ | ---------------------------------------------- |
| 400    | Malformed request or invalid filter.           |
| 401    | Missing or invalid API key.                    |
| 403    | Key lacks scope for the requested environment. |
| 409    | Idempotency key reuse with different body.     |
| 410    | Replay window (30 days) has elapsed.           |
| 429    | Publish rate limit exceeded — see `Retry-After`.|

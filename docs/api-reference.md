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
| `filter`      | string | no       | CEL expression over the event envelope.  |

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
| 409    | Idempotency key reuse with different body.   |
| 429    | Publish rate limit exceeded.                 |

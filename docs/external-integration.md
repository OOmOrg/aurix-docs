---
title: External Integration
description: Guide for integrating with external application via Webhooks and REST API.
icon: lucide/bow-arrow

---

# External Integration

Aurix can send event notifications to your specified endpoint using webhooks. This integration allows you to receive real-time updates for supported events directly to your backend system. Additionally, the Leads API allows integration partners to fetch batched lead data and attribution logs programmatically.

## Webhooks

Aurix sends webhook notifications as `POST` requests to your specified endpoint. Each request contains a JSON payload with information about the event that occurred.

### Supported Events

The webhooks support the following events:

*   **`widget.click`**: Triggers immediately when a visitor clicks the WhatsApp widget. Captures the "First Touch" attribution data.
*   **`lead.correlated`**: Triggers when an anonymous visitor sends their first message. This event links the `correlation_id` (from the click) to a specific phone number.
*   **`lead.complete`**: Triggers when a lead is marked as complete. This event sends the full lead details including name, email, and phone number collected by the AI agent.

### Configuration

To start receiving events, you must register your webhook URL via the API. This configuration endpoint is protected and requires your **Integration API Key**.

#### Register or Update URL

Registers or updates the webhook URL where you will receive events. Returns a `secret` used for signature verification.

**Endpoint:** `PUT /api/integrations/topkee/webhook-config`

```bash title="Request"
curl -X PUT \
  "https://chatbot.oomdigital.com/api/integrations/topkee/webhook-config" \
  -H "Authorization: Bearer <TOPKEE_INTEGRATION_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-backend.com/webhooks/aurix"
  }'
```

```json title="Response"
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "service_name": "topkee",
  "target_url": "https://your-backend.com/webhooks/aurix",
  "secret": "aurix_8f7d6e5c4b3a2...",
  "is_active": true
}
```

!!! warning "Important"
    Store the `secret` securely. You will need it to verify the `x-hub-signature` header of incoming webhook requests.

#### Get Current Configuration

Retrieves the current webhook configuration.

**Endpoint:** `GET /api/integrations/topkee/webhook-config`

```bash title="Request"
curl -X GET \
  "https://chatbot.oomdigital.com/api/integrations/topkee/webhook-config" \
  -H "Authorization: Bearer <TOPKEE_INTEGRATION_API_KEY>"
```

### Webhook Authentication

Webhook authentication ensures that incoming webhook requests are securely verified before processing. This allows consumers to trust that webhook events originate from a secure and verified source.

#### Verifying the Signature

Each webhook request sent from the server includes an `x-hub-signature` header containing a SHA-256 HMAC signature of the request payload.

1.  **Receive the Webhook**: Extract the payload and the `x-hub-signature` header.
2.  **Verify the Signature**:
    *   Compute the HMAC SHA-256 signature using the raw request body and the shared secret key.
    *   Compare the computed signature to the header value.
    *   If they match, the request is verified as authentic.

```python title="verification.py"
import hmac
import hashlib

def verify_signature(payload_body: str, signature_header: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode('utf-8'), 
        payload_body.encode('utf-8'), 
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(expected, signature_header)
```

### Event Reference & Payloads

Understanding the lead journey helps interpret the events:

1.  **Widget Interaction**: User clicks widget (`widget.click`).
2.  **Correlation**: User sends a message (`lead.correlated`). The lead is "incomplete" as the AI is still collecting info.
3.  **Completion**: AI collects required fields (`lead.complete`).

#### Common Schema

All events share the following top-level structure:

| Field | Type | Description |
| :--- | :--- | :--- |
| `event_type` | String | Name of the event type (e.g., `widget.click`). |
| `event_id` | String | Unique identifier for this specific event instance. |
| `timestamp` | String | ISO 8601 timestamp of when the event occurred. |
| `data` | Object | Event-specific payload containing attribution or lead details. |

#### widget.click

```json
{
  "event_type": "widget.click",
  "timestamp": "2024-01-23T10:30:00.123Z",
  "data": {
    "correlation_id": "O-xY123z",
    "device_id": "550e8400-e29b-41d4-a716-446655440000",
    "click_url": "https://example.com/landing?utm_source=google&gclid=AbCd123",
    "parsed_params": {
      "utm_source": "google",
      "utm_medium": "cpc",
      "gclid": "AbCd123"
    },
    "user_agent": "Mozilla/5.0...",
    "ip_address": "192.168.1.1" 
  }
}
```

#### lead.correlated

!!! note
    A correlated lead may initially have partial information while the AI engages the user.

```json
{
  "event_type": "lead.correlated",
  "timestamp": "2024-01-23T10:35:00.456Z",
  "data": {
    "correlation_id": "O-xY123z",
    "conversation_id": 105,
    "phone_number": "+1234567890",
    "lead_status": "correlated",
    "channel_id": "791163770738681"
  }
}
```

#### lead.complete

Fired when the AI has collected all required lead information.

```json
{
  "event_type": "lead.complete",
  "timestamp": "2024-01-23T10:45:00.789Z",
  "data": {
    "correlation_id": "O-xY123z",
    "conversation_id": 105,
    "lead_status": "complete",
    "lead_data": {
        "name": "John Doe",
        "email": "john.doe@example.com",
        "phone": "+1234567890",
        "company": "Acme Corp",
        "intent": "Interested in pricing"
    }
  }
}
```

---

## API Documentation

The Leads API allows integration partners to fetch batched lead data and attribution logs programmatically.

!!! note "Access Restricted"
    These endpoints are restricted to integration partners only. You must use the specific integration API key (`TOPKEE_INTEGRATION_API_KEY`) provided during onboarding.

### Authentication

All API requests must include your Integration API Key in the `Authorization` header.

```text
Authorization: Bearer <TOPKEE_INTEGRATION_API_KEY>
```

### Get Leads

Returns all records matching the date criteria.

**Endpoint:** `GET /api/integrations/topkee/leads`

**Query Parameters:**

*   `limit` (optional): Number of records to return.
*   `from_date` (optional): ISO 8601 String. Return records created after this date.
*   `to_date` (optional): ISO 8601 String. Return records created before this date.

```bash title="Request"
curl -X GET \
  "https://chatbot.oomdigital.com/api/integrations/topkee/leads?limit=10" \
  -H "Authorization: Bearer <TOPKEE_INTEGRATION_API_KEY>"
```

```json title="Response"
{
  "data": [
    {
      "wa_inbound_source_correlation_id": "O-xY123z",
      "conversation_id": 105,
      "created_at": "2024-01-23T10:30:00.123Z",
      "wa_inbound_source_correlated_at": "2024-01-23T10:35:00.456Z",
      "wa_inbound_source_click_url": "https://example.com/landing?...",
      "wa_inbound_source_parsed_params": {
        "utm_source": "google",
        "gclid": "AbCd123"
      }
    }
  ],
  "message": "Leads fetched successfully"
}
```

### Get Attribution Logs

Retrieves raw attribution logs (clicks), regardless of whether they converted to a lead. Returns a list of inbound source objects similar to the `widget.click` payload.

**Endpoint:** `GET /api/integrations/topkee/attribution`

```bash title="Request"
curl -X GET \
  "https://chatbot.oomdigital.com/api/integrations/topkee/attribution?limit=5" \
  -H "Authorization: Bearer <TOPKEE_INTEGRATION_API_KEY>"
```

### Error Codes

| Code | Description |
| :--- | :--- |
| **200** | Success |
| **401** | Unauthorized (Invalid or missing API Key) |
| **500** | Internal Server Error |


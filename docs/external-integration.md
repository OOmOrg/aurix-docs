---
title: External Integration
description: Guide for integrating with external application via Webhooks and REST API.
icon: lucide/bow-arrow
---

# Aurix External CDP Integration

Aurix can send event notifications to your specified endpoint using webhooks. This integration allows you to receive real-time updates for supported events directly to your backend system. This document serves as the technical reference for integrating with the Aurix platform. It covers authentication, webhook configuration, event schemas, and the Leads API.

For a step-by-step implementation guide, please refer to the [Integration Tutorial](tutorial-webhook-integration).

## API Overview

**Base URL**: `https://chatbot-api.oomdigital.com`

All API requests must be made over **HTTPS**.

### Authentication

Access to the Integration API is protected by a **Bearer Token**. You must include your `AURIX_API_KEY` (formerly `INTEGRATION_API_KEY`) in the `Authorization` header of every request. You can request for this key from the Aurix team.

```text
Authorization: Bearer <YOUR_AURIX_API_KEY>
```

---

## Webhooks

Webhooks allow your system to receive real-time notifications about lead activities and widget interactions.

### 1. Configuration

To start receiving events, you must register your webhook listener URL.

#### Register or Update URL

**Endpoint**: `PUT /integrations/external/webhook-config`

Registers or updates the URL where Aurix will send `POST` requests.

```bash title="Request"
curl -X PUT \
  "https://chatbot-api.oomdigital.com/integrations/external/webhook-config" \
  -H "Authorization: Bearer <YOUR_AURIX_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-backend.com/webhooks/aurix"
  }'
```

```json title="Response"
{
  "id": "dc2321f2-6a75-4d50-8861-3ea24ca29a84",
  "service_name": "external_cdp",
  "target_url": "https://your-backend.com/webhooks/aurix",
  "secret": "aurix_pzgwhW5X...",
  "is_active": true
}
```

!!! warning "Important"
    Save the `secret` from the response. You will need it to verify the `x-hub-signature` header on incoming events.

#### Get Configuration

**Endpoint**: `GET /integrations/external/webhook-config`

Retrieves your current webhook settings.

```bash title="Request"
curl -X GET \
  "https://chatbot-api.oomdigital.com/integrations/external/webhook-config" \
  -H "Authorization: Bearer <YOUR_AURIX_API_KEY>"
```

### 2. Event Payloads

Aurix sends JSON payloads for the following events.

#### Lead Lifecycle & Events

| Event | Status | Description | Action for CDP |
| :--- | :--- | :--- | :--- |
| `widget.click` | **Anonymous** | User clicked WhatsApp but hasn't messaged yet. | Create anonymous session/cookie profile. |
| `lead.correlated`| **Identified** | User sent a message; phone number is now linked to session. | Merge anonymous session with a User Profile. |
| `lead.complete` | **Qualified** | AI collected name, email, and intent. | Enrich profile and trigger CRM workflows. |

---

#### `widget.click`
Triggers when a visitor clicks the WhatsApp widget. Contains "First Touch" attribution.

For a detailed breakdown of what data fields are parsed and stored (e.g. UTM parameters, device info), please refer to the [Tracking Capabilities](index.md#current-vs-planned-tracking-capabilities) section.

*   `correlation_id`: (String) A unique session identifier used to link future events to this click.
*   `parsed_params`: (Object) Dictionary of UTM parameters and GCLID extracted from the URL.

```json
{
  "event_type": "widget.click",
  "event_id": "evt_1704200000",
  "timestamp": "2024-01-02T12:00:00Z",
  "data": {
    "correlation_id": "O-vpbWx3M",
    "device_id": "9fad5f62-2d08-49d4-852a-b48be43beb68",
    "click_url": "https://example.com/landing?utm_source=google",
    "parsed_params": {
      "utm_source": "google"
    },
    "ip_address": "127.0.0.1",
    "user_agent": "Mozilla/5.0..."
  }
}
```

#### `lead.correlated`
Triggers when an anonymous visitor sends their first message. Links the `correlation_id` to a phone number.

```json
{
  "event_type": "lead.correlated",
  "event_id": "evt_1704200500",
  "timestamp": "2024-01-02T12:05:00Z",
  "data": {
    "correlation_id": "O-vpbWx3M",
    "conversation_id": 3293,
    "phone_number": "6512345678",
    "lead_status": "correlated",
    "channel_id": "791163770738681"
  }
}
```

#### `lead.complete`
Triggers when the AI has collected all required information from the user (e.g., name, email, intent). This event signifies a qualified lead.

```json
{
  "event_type": "lead.complete",
  "event_id": "evt_1704201000",
  "timestamp": "2024-01-02T12:10:00Z",
  "data": {
    "correlation_id": "O-vpbWx3M",
    "conversation_id": 3293,
    "lead_status": "complete",
    "lead_data": {
      "name": "Mock User",
      "email": "mock@example.com",
      "phone": "+15550009999",
      "company": "Mock Inc",
      "intent": "Testing Webhooks"
    }
  }
}
```

### 3. Security & Reliability

#### Signature Verification
Every webhook request includes an `x-hub-signature` header. This is an **HMAC-SHA256** signature generated using your `secret`. **Always verify this signature** to ensure the request is genuine.

#### Retry Policy
Aurix currently uses a **Fire and Forget** mechanism. If your listener is down or returns a non-200 status code, the event **will not be retried**. Ensure your listener is highly available and responds quickly (within 10 seconds).

---

## Leads API

Fetch lead data and attribution logs programmatically.

### Get Leads

**Endpoint**: `GET /integrations/external/leads`

Returns a list of leads with their status, collected form data, and attribution info.

**Query Parameters**:
*   `limit`: (int) Max records to return (default 50).
*   `offset`: (int) Pagination offset.
*   `from_date`: (ISO 8601) Filter start date.
*   `to_date`: (ISO 8601) Filter end date.

```bash title="Request"
curl -X GET \
  "https://chatbot-api.oomdigital.com/integrations/external/leads?limit=5" \
  -H "Authorization: Bearer <YOUR_AURIX_API_KEY>"
```

```json title="Response"
{
  "status": 200,
  "message": "ok",
  "data": [
    {
      "lead_id": 470,
      "lead_status": "active",
      "lead_json": {
        "contact_name": "Jane Doe",
        "contact_number": "6587487282",
        "contact_email": "jane@example.com",
        "preferred_outlet": "Orchard Gateway",
        "selected_service": "Eyelash Extensions",
        "preferred_date_time": "1 January 2026"
      },
      "created_at": "2025-12-31T12:20:16.583181Z",
      "updated_at": "2026-01-02T01:38:05.868837Z",
      "click_url": null,
      "parsed_params": null,
      "ip_address": null,
      "user_agent": null
    }
  ]
}
```

### Get Attribution Logs

**Endpoint**: `GET /integrations/external/attribution`

Retrieves raw widget click logs, useful for auditing traffic sources.

```bash title="Request"
curl -X GET \
  "https://chatbot-api.oomdigital.com/integrations/external/attribution?limit=5" \
  -H "Authorization: Bearer <YOUR_AURIX_API_KEY>"
```

```json title="Response"
{
  "status": 200,
  "message": "ok",
  "data": [
    {
      "wa_inbound_source_correlation_id": "O-vpbWx3M",
      "wa_inbound_source_device_id": "9fad5f62-2d08-49d4-852a-b48be43beb68",
      "created_at": "2026-01-02T11:17:02.644566Z",
      "wa_inbound_source_click_url": "http://example.com/auth/login",
      "wa_inbound_source_first_raw_url": "http://example.com/auth/login",
      "wa_inbound_source_parsed_params": {},
      "wa_inbound_source_ip_address": "127.0.0.1",
      "wa_inbound_source_user_agent": "Mozilla/5.0..."
    }
  ]
}
```

---
title: "Integration Tutorial"
description: "Step-by-step guide to integrating with Aurix via Webhooks."
icon: "lucide/book-open"
---

# Step-by-Step Integration Guide

This guide will walk you through the end-to-end process of integrating with Aurix. You will learn how to set up a webhook listener, expose it to the internet, and register it with our API to receive real-time event notifications.

## Prerequisites

*   **API Key**: You should have received a `Bearer` token (API Key) from the Aurix team.
*   **Python Installed**: This guide uses Python for the example listener, but you can use any language.

---

## Step 1: Create a Webhook Listener

First, you need a server to receive the webhook `POST` requests. Below is a production-ready Python script using `FastAPI` that listens for events and verifies the security signature.

Create a file named `webhook_listener.py`:

```python
import uvicorn
from fastapi import FastAPI, Request, HTTPException, Header
import logging
import hmac
import hashlib
import os

# Configuration
# The secret you receive from Aurix after registering your URL
WEBHOOK_SECRET = os.environ.get("AURIX_WEBHOOK_SECRET", "your_shared_secret_here")

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("webhook_listener")

app = FastAPI()

def verify_signature(payload: bytes, signature_header: str, secret: str) -> bool:
    """Verifies the HMAC-SHA256 signature."""
    if not signature_header:
        return False
    
    # Calculate expected signature
    expected_signature = hmac.new(
        secret.encode("utf-8"),
        payload,
        hashlib.sha256
    ).hexdigest()
    
    return hmac.compare_digest(expected_signature, signature_header)

@app.post("/webhook")
async def receive_webhook(
    request: Request, 
    x_hub_signature: str = Header(None)
):
    # 1. Read the raw body
    raw_body = await request.body()
    
    # 2. Verify Signature
    # It is critical to verify the request came from Aurix
    if not verify_signature(raw_body, x_hub_signature, WEBHOOK_SECRET):
        logger.warning("Received request with invalid signature.")
        raise HTTPException(status_code=401, detail="Invalid signature")

    # 3. Parse and Process
    try:
        body = await request.json()
        event_type = body.get("event_type")
        
        logger.info(f"Received valid event: {event_type}")
        logger.info(f"Data: {body}")
        
        # Add your business logic here
        if event_type == "widget.click":
            # Handle click tracking
            pass
        elif event_type == "lead.correlated":
            # Handle new lead
            pass
            
        return {"status": "received"}
        
    except Exception as e:
        logger.error(f"Error processing webhook: {e}")
        raise HTTPException(status_code=500, detail="Internal processing error")

if __name__ == "__main__":
    print("Starting Webhook Listener on http://localhost:8005/webhook")
    uvicorn.run(app, host="0.0.0.0", port=8005)
```

### Install Dependencies

```bash
pip install fastapi uvicorn
```

### Run the Listener

```bash
python webhook_listener.py
```

---

## Step 2: Expose Your Listener

Aurix needs to reach your listener over the public internet. For development, you can use a tool like **ngrok** to create a secure tunnel to your localhost.

1.  **Start ngrok**:
    ```bash
    ngrok http 8005
    ```
2.  **Copy the URL**: ngrok will give you a public HTTPS URL (e.g., `https://a1b2-c3d4.ngrok.io`).
3.  **Append the path**: Your full webhook URL will be `https://a1b2-c3d4.ngrok.io/webhook`.

---

## Step 3: Register Your Webhook

Now you must tell Aurix where to send the events. You will use the **Configuration API** to set your URL.

**Endpoint**: `PUT https://chatbot-api.oomdigital.com/integrations/topkee/webhook-config`

**Command**:

Replace `<YOUR_API_KEY>` with the key provided to you, and `<YOUR_WEBHOOK_URL>` with your public URL from Step 2.

```bash
curl -X PUT "https://chatbot-api.oomdigital.com/integrations/topkee/webhook-config" \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer <YOUR_API_KEY>" \
     -d '{
           "url": "<YOUR_WEBHOOK_URL>"
         }'
```

### Save the Secret!

The API will respond with a JSON object containing a `secret`.

```json
{
  "id": "...",
  "service_name": "topkee",
  "target_url": "https://a1b2-c3d4.ngrok.io/webhook",
  "secret": "aurix_8f7d9a8b...", 
  "is_active": true
}
```

**Copy the `secret` value.** Stop your python script, set this as an environment variable (or update the code), and restart the listener. This secret is required to pass the signature verification in Step 1.

```bash
export AURIX_WEBHOOK_SECRET="aurix_8f7d9a8b..."
python webhook_listener.py
```

---

## Step 4: Receive Events

Once registered, Aurix will immediately begin sending events to your URL.

### Example: Widget Click Event

When a user clicks the WhatsApp widget, your listener will receive:

```json
{
  "event_type": "widget.click",
  "event_id": "evt_1704200000",
  "timestamp": "2024-01-02T12:00:00Z",
  "data": {
    "correlation_id": "wa_123456",
    "click_url": "https://example.com/landing?utm_source=google",
    "parsed_params": {
      "utm_source": "google"
    },
    "ip_address": "1.2.3.4",
    "user_agent": "Mozilla/5.0..."
  }
}
```

### Example: Lead Correlated Event

When that user sends a message and is identified:

```json
{
  "event_type": "lead.correlated",
  "event_id": "evt_1704200500",
  "timestamp": "2024-01-02T12:05:00Z",
  "data": {
    "correlation_id": "wa_123456",
    "conversation_id": 987,
    "phone_number": "+15551234567",
    "lead_status": "correlated",
    "channel_id": "..."
  }
}
```

You have now successfully integrated with Aurix!


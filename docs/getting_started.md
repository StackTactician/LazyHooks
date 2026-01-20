# Getting Started

## Installation

Install LazyHooks via pip:

```bash
pip install lazyhooks
```

## Basic Usage

### Sending a Webhook

The simplest way to send a webhook is "Fire & Forget". LazyHooks handles the signing and sending.

```python
import asyncio
from lazyhooks import WebhookSender

async def main():
    # Initialize sender with your secret key
    sender = WebhookSender(signing_secret="super-secret")
    
    # Send the webhook
    # This automatically adds specific `X-Lh-Timestamp` and `X-Lh-Signature` headers
    await sender.send(
        url="https://example.com/webhook",
        payload={
            "event": "user.created", 
            "user_id": 123,
            "timestamp": "2023-10-27T10:00:00Z"
        }
    )

asyncio.run(main())
```

### Receiving a Webhook

LazyHooks provides a helper to verify the HMAC signature and timestamp of incoming webhooks.

```python
from lazyhooks import verify_signature

# Example using a generic web framework handler
def handle_incoming_webhook(request):
    signature = request.headers.get("X-Lh-Signature")
    timestamp = request.headers.get("X-Lh-Timestamp")
    body_bytes = request.body
    
    is_valid = verify_signature(
        payload_body=body_bytes, 
        signature_header=signature, 
        secret="super-secret",
        timestamp_header=timestamp
    )
    
    if is_valid:
        return "Webhook Verified!", 200
    else:
        return "Invalid Signature", 401
```

[Next: Comparisons - Why LazyHooks?](./comparisons.md)

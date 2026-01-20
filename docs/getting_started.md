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

LazyHooks provides a helper to verify the HMAC signature of incoming webhooks.

```python
from lazyhooks import verify_signature

# Example using a generic web framework handler
def handle_incoming_webhook(request):
    # 1. Get the signature from headers
    signature = request.headers.get("X-Hub-Signature-256")
    
    # 2. Get the raw body bytes
    body_bytes = request.body
    
    # 3. Verify
    is_valid = verify_signature(
        payload_body=body_bytes, 
        signature_header=signature, 
        secret="super-secret"
    )
    
    if is_valid:
        return "Webhook Verified!", 200
    else:
        return "Invalid Signature", 401
```

[Next: Comparisons - Why LazyHooks?](./comparisons.md)

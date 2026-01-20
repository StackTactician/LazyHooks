# Comparisons: Why LazyHooks?

This guide demonstrates how LazyHooks simplifies webhook logic by comparing the "manual way" (Without LazyHooks) versus the "easy way" (With LazyHooks).

We'll look at three scenarios:
1. **FastAPI** (Web Framework)
2. **Django** (Web Framework)
3. **Celery** (Task Queue)

---

## 1. FastAPI: Standard Webhook Receiver

The manual approach requires repetitive HMAC verification code.

### WITHOUT LazyHooks

```python
import hmac
import hashlib
from fastapi import FastAPI, Request, HTTPException

app = FastAPI()
SECRET = "my_secret_key"

@app.post("/webhook")
async def manual_receiver(request: Request):
    body = await request.body()
    signature_header = request.headers.get("X-Hub-Signature-256")
    
    if not signature_header or not signature_header.startswith("sha256="):
        raise HTTPException(403, "Missing signature")

    # Manually compute HMAC
    expected = hmac.new(SECRET.encode(), body, hashlib.sha256).hexdigest()
    incoming = signature_header.split("=")[1]
    
    # Secure comparison to prevent timing attacks
    if not hmac.compare_digest(expected, incoming):
        raise HTTPException(403, "Invalid signature")

    return {"status": "verified"}
```

### WITH LazyHooks

LazyHooks handles the extraction, computation, and secure comparison in one line.

```python
from fastapi import FastAPI, Request, HTTPException
from lazyhooks import verify_signature

app = FastAPI()
SECRET = "my_secret_key"

@app.post("/webhook")
async def rapid_receiver(request: Request):
    body = await request.body()
    signature = request.headers.get("X-Hub-Signature-256")
    
    # One-liner verification
    if not verify_signature(body, signature, SECRET):
        raise HTTPException(403, "Invalid signature")

    return {"status": "verified"}
```

---

## 2. Django: Dealing with bytes and strings

Django request bodies can be tricky to handle correctly for signature verification.

### WITHOUT LazyHooks

```python
import hmac
import hashlib
from django.http import HttpResponse, HttpResponseForbidden
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def manual_view(request):
    if request.method != "POST":
         return HttpResponse("Method not allowed", status=405)

    signature = request.headers.get("X-Hub-Signature-256")
    if not signature:
         return HttpResponseForbidden("Missing signature")

    # Manually construct expected signature
    expected = hmac.new(
        b"my_secret_key", 
        request.body, 
        hashlib.sha256
    ).hexdigest()

    incoming = signature.split("=")[1]
    
    if not hmac.compare_digest(expected, incoming):
        return HttpResponseForbidden("Invalid signature")

    return HttpResponse("Verified")
```

### WITH LazyHooks

```python
from django.http import HttpResponse, HttpResponseForbidden
from django.views.decorators.csrf import csrf_exempt
from lazyhooks import verify_signature

@csrf_exempt
def rapid_view(request):
    signature = request.headers.get("X-Hub-Signature-256")
    
    if not verify_signature(request.body, signature, "my_secret_key"):
        return HttpResponseForbidden("Invalid signature")

    return HttpResponse("Verified")
```

---

## 3. Celery: Sending Retriable Webhooks

Sending webhooks reliably often involves task queues like Celery. However, Celery requires a full broker (Redis/RabbitMQ) infrastructure.

### WITHOUT LazyHooks (Using Celery)

*Prerequisite: Redis/RabbitMQ running, Celery worker process running.*

```python
# tasks.py
from celery import Celery
import requests
import hmac
import hashlib
import json

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task(bind=True, max_retries=3)
def send_webhook_task(self, url, payload):
    # 1. Manual Signing
    payload_bytes = json.dumps(payload).encode()
    signature = hmac.new(b"secret", payload_bytes, hashlib.sha256).hexdigest()
    
    headers = {"X-Hub-Signature-256": f"sha256={signature}"}
    
    try:
        # 2. Sending
        resp = requests.post(url, data=payload_bytes, headers=headers, timeout=10)
        resp.raise_for_status()
    except Exception as exc:
        # 3. Manual Retry Logic
        raise self.retry(exc=exc, countdown=60)
```

### WITH LazyHooks

LazyHooks provides persistent retries **without** needing a separate broker or heavy worker process.

```python
import asyncio
from lazyhooks import WebhookSender
from lazyhooks.storage.sqlite import SQLiteStorage

# Zero infrastructure retries
storage = SQLiteStorage("webhooks.db")
sender = WebhookSender("secret", storage=storage)

# Just run this background task in your main app
asyncio.create_task(sender.retry_worker())

# Send - if it fails, it's saved to SQLite and retried automatically
await sender.send("https://example.com/webhook", {"event": "hello"})
```

---
[Next: API Reference](./api_reference.md)

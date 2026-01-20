# Comparisons: Why LazyHooks?

This guide demonstrates how LazyHooks simplifies webhook logic by comparing the "manual way" (Without LazyHooks) versus the "easy way" (With LazyHooks).

We'll look at three scenarios:
1. **FastAPI** (Web Framework)
2. **Django** (Web Framework)
3. **Celery** (Task Queue)

---

## 1. FastAPI: Standard Webhook Receiver

The manual approach requires verifying the signature **and** checking the timestamp freshness.

### WITHOUT LazyHooks

```python
import hmac
import hashlib
import time
from fastapi import FastAPI, Request, HTTPException

app = FastAPI()
SECRET = "my_secret_key"

@app.post("/webhook")
async def manual_receiver(request: Request):
    body = await request.body()
    signature_header = request.headers.get("X-Lh-Signature")
    timestamp_header = request.headers.get("X-Lh-Timestamp")
    
    if not signature_header or not timestamp_header:
        raise HTTPException(403, "Missing headers")

    try:
        timestamp = int(timestamp_header)
    except ValueError:
        raise HTTPException(403, "Invalid timestamp")

    if int(time.time()) - timestamp > 300: # 5 min tolerance
        raise HTTPException(403, "Request too old")

    to_sign = f"{timestamp}.".encode() + body
    
    expected = hmac.new(SECRET.encode(), to_sign, hashlib.sha256).hexdigest()
    incoming = signature_header.split("v1=")[1]
    
    if not hmac.compare_digest(expected, incoming):
        raise HTTPException(403, "Invalid signature")

    return {"status": "verified"}
```

### WITH LazyHooks

LazyHooks handles timestamp checking, string reconstruction, and secure comparison in one line.

```python
from fastapi import FastAPI, Request, HTTPException
from lazyhooks import verify_signature

app = FastAPI()
SECRET = "my_secret_key"

@app.post("/webhook")
async def rapid_receiver(request: Request):
    body = await request.body()
    signature = request.headers.get("X-Lh-Signature")
    timestamp = request.headers.get("X-Lh-Timestamp")
    
    # One-liner verification (includes timestamp check)
    if not verify_signature(body, signature, SECRET, timestamp):
        raise HTTPException(403, "Invalid signature")

    return {"status": "verified"}
```

---

## 2. Django: Dealing with bytes and strings

### WITHOUT LazyHooks

```python
import hmac
import hashlib
import time
from django.http import HttpResponse, HttpResponseForbidden
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def manual_view(request):
    if request.method != "POST":
         return HttpResponse("Method not allowed", status=405)

    signature_header = request.headers.get("X-Lh-Signature")
    timestamp_header = request.headers.get("X-Lh-Timestamp")

    if not signature_header or not timestamp_header:
         return HttpResponseForbidden("Missing headers")
         
    # Timestamp check...
    if int(time.time()) - int(timestamp_header) > 300:
        return HttpResponseForbidden("Expired")

    to_sign = f"{timestamp_header}.".encode() + request.body
    
    expected = hmac.new(b"my_secret_key", to_sign, hashlib.sha256).hexdigest()
    incoming = signature_header.split("v1=")[1]
    
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
    signature = request.headers.get("X-Lh-Signature")
    timestamp = request.headers.get("X-Lh-Timestamp")
    
    if not verify_signature(request.body, signature, "my_secret_key", timestamp):
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
import time

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task(bind=True, max_retries=3)
def send_webhook_task(self, url, payload):
    # 1. Manual Signing (Complex!)
    payload_bytes = json.dumps(payload).encode()
    timestamp = int(time.time())
    to_sign = f"{timestamp}.".encode() + payload_bytes
    
    signature = hmac.new(b"secret", to_sign, hashlib.sha256).hexdigest()
    
    headers = {
        "X-Lh-Signature": f"v1={signature}",
        "X-Lh-Timestamp": str(timestamp)
    }
    
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

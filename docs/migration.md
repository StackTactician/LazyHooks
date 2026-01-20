# Migrating to LazyHooks

This guide helps you upgrade from manual webhook implementations (using `requests` or `aiohttp`) to LazyHooks.

## Why Migrate?

| Feature | Manual Implementation | LazyHooks |
|---------|----------------------|-----------|
| **Security** | ❌ Manual HMAC signing | ✅ Automatic signing & timestamping |
| **Reliability** | ❌ Basic or no retries | ✅ Smart retries & exponential backoff |
| **Visibility** | ❌ Print statements | ✅ Structured logging & metrics |
| **Persistence** | ❌ Lost on restart | ✅ Redis/SQLite persistence |

---

## 1. Migrating from `requests`

If you are currently sending webhooks like this:

### Before (Manual)

```python
import requests
import hmac
import hashlib
import json
import time

def send_webhook(url, data, secret):
    # 1. Manually serialize
    payload = json.dumps(data)
    
    # 2. Manually sign
    timestamp = str(int(time.time()))
    sig = hmac.new(
        secret.encode(), 
        f"{timestamp}.{payload}".encode(), 
        hashlib.sha256
    ).hexdigest()
    
    # 3. Send (blocking)
    try:
        requests.post(url, data=payload, headers={
            "X-Signature": sig,
            "X-Timestamp": timestamp
        }, timeout=10)
    except Exception as e:
        print(f"Failed: {e}")
        # Retries? Maybe later...
```

### After (LazyHooks)

Use `SyncWebhookSender` or `presets.production` for an instant upgrade.

```python
from lazyhooks import presets

# One-time setup
sender = presets.production(
    signing_secret="my-secret",
    redis_url="redis://localhost:6379/0"
)

# Usage
sender.send_sync("https://example.com/hook", data)
```

**Improvements:**
- **Automatic Retries**: If the server is down (5xx) or rate limited (429), LazyHooks retries automatically.
- **Persistence**: If your app crashes, the webhook is saved in Redis and sent later.
- **Standard Headers**: Uses standard `X-Lh-Signature` headers.

---

## 2. Migrating from `aiohttp`

### Before (Manuel Async)

```python
async with aiohttp.ClientSession() as session:
    await session.post(url, json=data)
    # Hope it didn't fail...
```

### After (LazyHooks Async)

```python
from lazyhooks import presets

sender = presets.production("secret", "redis://...")

await sender.send(url, data)
```

---

## 3. Adopting Best Practices

### Use Presets
Don't guess configuration values. Start with a preset:

- **Local Dev**: `presets.development()`
- **Production**: `presets.production()`
- **High Scale**: `presets.high_volume()`

### Better Error Handling
LazyHooks gives you specific errors to handle:

```python
from lazyhooks.exceptions import WebhookRateLimitError

try:
    sender.send_sync(url, data)
except WebhookRateLimitError as e:
    print(f"Rate limited! Retry after {e.retry_after}s")
```

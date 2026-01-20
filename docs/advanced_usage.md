# Advanced Usage

## Persistence & Retries

LazyHooks can use a storage backend to persist webhooks. This ensures that if a delivery fails (e.g., the receiver is down), it is safely stored and retried later.

### Using SQLite Storage

```python
import asyncio
from lazyhooks import WebhookSender
from lazyhooks.storage.sqlite import SQLiteStorage

async def main():
    # 1. Initialize Storage
    storage = SQLiteStorage("webhooks.db")
    
    # 2. Initialize Sender with storage
    sender = WebhookSender(signing_secret="my-secret", storage=storage)
    
    # 3. Send a webhook
    # If this fails, it is saved to 'webhooks.db' with status='failed'
    await sender.send("https://example.com/endpoint", {"data": "important"})
    
    # 4. Start the Retry Worker (usually run this as a background task)
    # This worker polls the DB for failed events and retries them
    asyncio.create_task(sender.retry_worker())
    
    # Keep the process alive for demo purposes
    await asyncio.sleep(60)

asyncio.run(main())
```

**Retry Schedule:**
LazyHooks uses a default exponential backoff:
1. 1 minute
2. 5 minutes
3. 30 minutes
4. 1 hour

After the last attempt, the status is marked as `dead`.

---

## Custom Configuration

### Custom Headers

You can pass custom headers to the `send` method. These will be included in the HTTP request alongside the default signature headers.

```python
await sender.send(
    url="...",
    payload={...},
    headers={"X-Custom-ID": "12345", "Authorization": "Bearer token"}
)
```

### Timeouts

The default timeout is 10 seconds. You can override this globally or per-request.

**Global Timeout:**
```python
sender = WebhookSender("secret", default_timeout=5.0)
```

**Per-Request Timeout:**
```python
await sender.send(url="...", payload={...}, timeout=30.0)
```

---

## Scheduling

You can schedule a webhook to be sent in the future using the `schedule_at` parameter. **Note:** Storage is required for scheduling.

```python
import time

future_time = time.time() + 3600 # 1 hour from now

await sender.send(
    url="...", 
    payload={...}, 
    schedule_at=future_time
)
```

## Synchronous Usage

If you are working in a synchronous context (like a standard Django view or a script), you can use `send_sync`.

> **Warning:** Do not use `send_sync` if you are already inside an `asyncio` event loop.

```python
sender = WebhookSender("secret")

# Blocks until sent (or queued)
sender.send_sync("https://example.com/webhook", {"msg": "hello sync"})
```

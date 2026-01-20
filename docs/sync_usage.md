# Synchronous vs Asynchronous Usage

LazyHooks supports both Synchronous (blocking) and Asynchronous (non-blocking) execution.

## Which one should I use?

| Use Case | Recommended API |
|----------|-----------------|
| **Django** (standard views) | `SyncWebhookSender` |
| **Flask** (standard views) | `SyncWebhookSender` |
| **Scripts / Automation** | `SyncWebhookSender` |
| **FastAPI / Quart** | `WebhookSender` (Async) |
| **High Performance / Bulk** | `WebhookSender` (Async) |

## Asynchronous API (Default)

Use `WebhookSender` with `async/await`.

```python
import asyncio
from lazyhooks import WebhookSender

async def job():
    sender = WebhookSender("secret")
    await sender.send("https://example.com/hook", {"event": "test"})

asyncio.run(job())
```

## Synchronous API

Use `SyncWebhookSender` which handles the event loop for you.

```python
from lazyhooks import SyncWebhookSender

def job():
    # Looks just like the async version, but no await!
    sender = SyncWebhookSender("secret")
    
    sender.send("https://example.com/hook", {"event": "test"})

# Run standard sync code
if __name__ == "__main__":
    job()
```

### Context Managers

Both support context managers for cleaner resource management.

```python
with SyncWebhookSender("secret") as sender:
    sender.send(url, data)
```

## Sync Receiver

For standard sync web frameworks, use `process_event_sync`.

```python
# Flask Example
@app.route('/webhooks', methods=['POST'])
def handle_webhook():
    # ... verify signature ...
    receiver.process_event_sync(event_data)
    return "OK"
```

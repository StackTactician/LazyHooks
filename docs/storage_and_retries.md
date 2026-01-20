# Persistence, Retries & Scheduling

LazyHooks can use a storage backend to promote reliability. If a delivery fails, it is saved and retried.

## Setup SQLite Storage

```python
import asyncio
from lazyhooks import WebhookSender
from lazyhooks.storage.sqlite import SQLiteStorage

async def main():
    storage = SQLiteStorage("webhooks.db")
    sender = WebhookSender("secret", storage=storage)
    
    # If this fails, it is saved to DB
    await sender.send("https://example.com/endpoint", {"data": "important"})
    
    # Start the background retry worker
    asyncio.create_task(sender.retry_worker())
    
    await asyncio.sleep(60) # Keep app alive
```

## Using Redis Storage (Recommended for Production)

For distributed systems or higher volume, use Redis.

```python
# Configure using a Redis URL
sender = WebhookSender(
    signing_secret="secret",
    storage="redis://localhost:6379/0"
)
```

Requires `pip install redis`.

## Retry Schedule
Default exponential backoff:
1. 1 minute
2. 5 minutes
3. 30 minutes
4. 1 hour

## Scheduling

You can schedule a webhook for the future (requires storage).

```python
import time
future = time.time() + 3600 # 1 hour later

await sender.send(
    url="...",
    payload={...},
    schedule_at=future
)
```

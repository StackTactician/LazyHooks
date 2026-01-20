# Sending Webhooks

LazyHooks makes it easy to send signed webhooks.

## Asynchronous Usage (Recommended)

The standard way to send webhooks uses `async/await`. This is non-blocking and ideal for high-throughput applications.

```python
import asyncio
from lazyhooks import WebhookSender

async def main():
    sender = WebhookSender(signing_secret="super-secret")
    
    # Non-blocking send
    await sender.send(
        url="https://example.com/webhook",
        payload={"event": "user.created", "id": 123}
    )

asyncio.run(main())
```

## Synchronous Usage

If you are not using `asyncio` (e.g., standard Django views, Flask 1.x, or simple scripts), use the `send_sync` method.

> **Warning:** Do not use `send_sync` if you are already inside an running `asyncio` loop (like FastAPI), as it will raise a RuntimeError. Use `await sender.send()` instead.

```python
from lazyhooks import WebhookSender

sender = WebhookSender("secret")

# Blocks execution until sent (or queued)
sender.send_sync("https://example.com/webhook", {"id": 123})
```

## Custom Headers & Timeouts

You can pass additional options to both `send` and `send_sync`.

```python
await sender.send(
    url="...",
    payload={...},
    headers={"Authorization": "Bearer token"},
    timeout=30.0 # Seconds
)
```

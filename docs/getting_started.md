# Getting Started

## Installation

```bash
pip install lazyhooks
```

## Documentation Index

- **[Sending Webhooks](sending_webhooks.md)**: Fire & forget, simple sending.
- **[Receiving Webhooks](receiving_webhooks.md)**: Verify signatures, parse events, and handle routing.
- **[Storage & Retries](storage_and_retries.md)**: SQLite persistence, background retry workers, and scheduling.
- **[Security](security.md)**: Details on HMAC signatures and reply protection.
- **[Comparisons](comparisons.md)**: Why use LazyHooks?
- **[API Reference](api_reference.md)**: Full API details.

## Quick Example (Send & Receive)

```python
# Sender
sender = WebhookSender("secret")
await sender.send("http://localhost:5000", {"event": "hello"})

# Receiver
receiver = WebhookReceiver("secret")
@receiver.on("hello")
async def on_hello(event):
    print("Received!", event)
```

# LazyHooks

A lightweight, standalone Python package for sending and receiving webhooks with optional persistence.

>**[Full Documentation & Examples on GitHub](https://github.com/StackTactician/LazyHooks)**

## Features

- **Simple API**: Send webhooks with minimal boilerplate.
- **Async First**: Built on `asyncio` and `aiohttp` for high performance.
- **Secure**: Built-in HMAC-SHA256 signing and verification.
- **Reliable**: Optional `SQLite` storage to persist and retry failed webhooks.

## Documentation

- **[Getting Started](docs/getting_started.md)**: Installation and basic usage.
- **[Comparisons](docs/comparisons.md)**: See LazyHooks vs standard FastAPI/Django/Celery.
- **[Advanced Usage](docs/advanced_usage.md)**: Persistence, retries, headers, and timeouts.
- **[API Reference](docs/api_reference.md)**: Detailed API documentation.

## Quick Example

```python
import asyncio
from lazyhooks import WebhookSender

async def main():
    sender = WebhookSender(signing_secret="super-secret")
    await sender.send("https://example.com/webhook", {"event": "hello"})

asyncio.run(main())
```

For receiving webhooks and more examples, check the [Getting Started](docs/getting_started.md) guide.

## Links

- **GitHub**: [https://github.com/StackTactician/LazyHooks](https://github.com/StackTactician/LazyHooks)
- **Issues**: [https://github.com/StackTactician/LazyHooks/issues](https://github.com/StackTactician/LazyHooks/issues)
- **Documentation**: See the GitHub repo for full docs, advanced usage, and examples.


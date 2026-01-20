# Receiving Webhooks

LazyHooks provides a `WebhookReceiver` class to simplify handling incoming webhooks. It handles signature verification, JSON parsing, error handling, and routing automatically.

## Quick Start (Async)

```python
from lazyhooks import WebhookReceiver

receiver = WebhookReceiver(signing_secret="my-super-secret-key")

@receiver.on("user.created")
async def handle_user_created(event):
    print(f"New user: {event.get('user_id')}")

# Run simple standalone server
if __name__ == "__main__":
    receiver.run(port=5000)
```

## Synchronous Usage

If you are using a synchronous framework (like standard Django or Flask without async routes) or writing a script, `WebhookReceiver` supports synchronous handlers and execution.

### Synchronous Handlers
You can use standard `def` functions as handlers.

```python
@receiver.on("order.created")
def handle_order(event):
    # This runs synchronously
    print("Order via sync handler:", event)
```

### Synchronous Processing (`process_event_sync`)
If you need to trigger event processing from synchronous code, use `process_event_sync`. This ensures async middleware and logic run correctly.

```python
# Standard request handler
def my_view(request):
    event_data = {"event": "test"} # ... verify and parse ...
    
    # Blocks until all handlers verify/complete
    receiver.process_event_sync(event_data)
    
    return "OK"
```

## Asynchronous Usage

Asynchronous usage is recommended for high-performance applications using `FastAPI`, `Quart`, or async Django views.

### Asynchronous Handlers
Use `async def` for handlers to perform non-blocking I/O.

```python
@receiver.on("payment.success")
async def handle_payment(event):
    await db.save_payment(event)
    print("Async payment saved")
```

### Asynchronous Processing (`process_event`)
Use `await process_event()` in async contexts.

```python
async def my_async_view(request):
    event = await request.json()
    await receiver.process_event(event) # Non-blocking
    return {"status": "ok"}
```

## Features

### Event Routing
Use the `@receiver.on(event_name)` decorator. You can also use `*` as a wildcard or catch-all.

```python
@receiver.on("order.*")
async def handle_orders(event):
    print("Order event:", event)

@receiver.on("*")
def log_all(event):
    print("Received:", event)
```

### Middleware
Middleware allows you to run code before/after handlers. Middleware MUST be `async def`.

```python
@receiver.middleware
async def logging_middleware(event, next_handler):
    print("Processing event...")
    # next_handler returns an awaitable even for sync handlers
    result = await next_handler(event) 
    print("Done.")
    return result
```

### Pydantic Validation
If `pydantic` is installed, you can pass a schema model for automatic validation.

```python
from pydantic import BaseModel

class UserEvent(BaseModel):
    event: str
    user_id: int

@receiver.on("user.created", schema=UserEvent)
async def handle_user(event: UserEvent):
    # event is a UserEvent object
    print(event.user_id)
```

## Framework Integration

### Flask (Sync)
```python
@app.route('/webhooks', methods=['POST'])
def webhook_view():
    # Use sync methods
    # ... verify manually or use adapter ...
    pass
```
*Note: The built-in `as_flask_view()` returns an async handler, which requires Flask 2.0+.*

### FastAPI (Async)
```python
app.post("/webhooks")(receiver.as_fastapi_endpoint())
```

### Django (Async Views)
```python
path('webhooks/', receiver.as_django_view(), name='webhooks'),
```

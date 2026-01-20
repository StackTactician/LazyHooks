# Testing LazyHooks

LazyHooks provides a `lazyhooks.testing` module to make testing your webhook processing easier.

## Installation

```bash
pip install lazyhooks[testing]
```

## Helpers

### `create_webhook_request`

Use this to generate properly signed payloads (with headers) for testing your receivers.

```python
from lazyhooks.testing import create_webhook_request

def test_my_receiver():
    # Simulate an event
    req = create_webhook_request(
        data={"event": "user.created", "id": 123},
        signing_secret="my-secret"
    )
    
    # req['json'] is the payload
    # req['headers'] contains X-Lh-Signature and X-Lh-Timestamp
    
    # Pass to your app (example with generic client)
    # client.post("/webhooks", json=req['json'], headers=req['headers'])
```

### `MockWebhookSender`

Use this to test your logic without actually sending network requests.

```python
from lazyhooks.testing import MockWebhookSender

async def test_my_logic():
    sender = MockWebhookSender()
    
    await sender.send("http://example.com", {"foo": "bar"})
    
    assert sender.call_count == 1
    assert sender.last_call['payload'] == {"foo": "bar"}
```

## Pytest Fixtures

If you use `pytest`, these fixtures are automatically available:

- `mock_webhook_sender`: An instance of `MockWebhookSender`.
- `signed_payload_factory`: Alias for `create_webhook_request`.

```python
async def test_with_fixtures(mock_webhook_sender):
    await mock_webhook_sender.send("...", {})
    assert mock_webhook_sender.call_count == 1
```

## mocking HTTP requests

For more advanced testing, we recommend using `responses` (included in `testing` extra).

```python
import responses

@responses.activate
async def test_network_call():
    responses.add(responses.POST, "http://hooks.com", status=200)
    # ... use real WebhookSender ...
```

# Error Handling

LazyHooks provides a rich exception hierarchy to help you handle errors precisely.

## Exception Hierarchy

All exceptions inherit from `lazyhooks.exceptions.WebhookError`.

- `WebhookError`
  - `WebhookConfigurationError`: Bad config.
  - `WebhookDeliveryError`: Base for delivery failures.
    - `WebhookNetworkError`: DNS, Connection Refused.
      - `WebhookTimeoutError`: Request timed out.
    - `WebhookHTTPError`: Non-200 responses.
      - `WebhookClientError` (4xx)
        - `WebhookRateLimitError` (429) - **Retryable**
        - `WebhookBadRequestError` (400)
        - `WebhookUnauthorizedError` (401)
        - `WebhookNotFoundError` (404)
      - `WebhookServerError` (5xx) - **Retryable**
  - `WebhookVerificationError`
    - `InvalidSignatureError`
    - `ExpiredTimestampError`

## Handling Errors

```python
from lazyhooks import WebhookSender
from lazyhooks.exceptions import (
    WebhookRateLimitError, 
    WebhookServerError,
    WebhookClientError
)

sender = WebhookSender("secret")

try:
    await sender.send(url, data)
except WebhookRateLimitError as e:
    print(f"Rate limited! Retry after {e.retry_after}s")
    # You might want to retry here manually or let LazyHooks storage handle it
except WebhookClientError as e:
    print(f"Client error {e.status_code}: {e.message}")
    # Don't retry 4xx errors
except WebhookServerError as e:
    print("Server error, retrying...")
```

## Retry Logic

`WebhookSender` handles retries automatically for network and server errors if storage is configured. 

If storage is **not** configured, `send()` will try once and implement basic in-memory retries for `lazyhooks.exceptions` marked as retryable (network/server/ratelimit) BUT if these fail, the exception will bubble up to you so you can handle it.

# API Reference

## WebhookSender

`lazyhooks.sender.WebhookSender`

### `__init__(signing_secret, storage=None, default_timeout=10.0)`

- **signing_secret** (`str`): The secret key used to sign the payload (HMAC-SHA256).
- **storage** (`BaseStorage`, optional): A storage backend instance (e.g., `SQLiteStorage`). Required for persistence and scheduling.
- **default_timeout** (`float`, default=10.0): Default connection/read timeout in seconds.

### `async send(url, payload, schedule_at=None, headers=None, timeout=None)`

Sends a webhook to the specified URL.

- **url** (`str`): The destination URL.
- **payload** (`dict`): The JSON payload to send.
- **schedule_at** (`float`, optional): Unix timestamp to schedule delivery for later. Requires `storage`.
- **headers** (`dict`, optional): Additional headers to include.
- **timeout** (`float`, optional): Request timeout in seconds. Overrides `default_timeout`.

**Returns:** `str` (Event ID).

### `async retry_worker()`

A background coroutine that polls the storage for pending/failed events and retries them. Should be run as an `asyncio.Task`.

---

## WebhookReceiver

`lazyhooks.receiver.WebhookReceiver`

### `__init__(signing_secret, tolerance=300, on_error=None, return_errors=False, enable_metrics=False)`

- **signing_secret** (`str`): The secret key used to verify signatures.
- **tolerance** (`int`, default=300): Max signature age in seconds.
- **on_error** (`callable`, optional): Callback for exceptions.
- **return_errors** (`bool`, default=False): If True, re-raises exceptions instead of logging them.

### `on(event_type, schema=None)`

Decorator to register an event handler.

- **event_type** (`str`): The event name to match (or `*` for catch-all).
- **schema** (`pydantic.BaseModel`, optional): Data model for validation.

### `middleware(func)`

Decorator to register middleware.

### `async process_event(event_data)`

Processes a parsed event dict through middleware and handlers.

### `process_event_sync(event_data)`

Synchronous wrapper for `process_event`. Uses `asyncio.run()`.

### `verify_and_parse(body, signature, timestamp)`

Verifies signature and parses JSON body. Sync method. Returns `dict`.

---


`lazyhooks.receiver.verify_signature`

### `verify_signature(payload_body, signature_header, secret, timestamp_header, tolerance=300)`

Verifies that the signature matches the payload AND that the request is fresh (default 5 minutes).

- **payload_body** (`bytes`): The raw request body bytes.
- **signature_header** (`str`): The content of the `X-Lh-Signature` header (expected format: `v1=<hash>`).
- **secret** (`str`): The shared signing secret.
- **timestamp_header** (`str`): The content of the `X-Lh-Timestamp` header.
- **tolerance** (`int`, default=300): The maximum allowed age of the request in seconds.

**Returns:** `bool` (`True` if valid, `False` otherwise).

---

## Storage

`lazyhooks.storage.sqlite.SQLiteStorage`

### `__init__(db_path="webhooks.db")`

Initializes a SQLite-backed storage.

- **db_path** (`str`): Path to the SQLite database file.

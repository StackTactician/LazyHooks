# Monitoring and Observability

LazyHooks includes optional integrations with popular monitoring tools. It automatically detects and uses installed libraries.

## Installation

Install the integrations you need:

```bash
# Metrics
pip install lazyhooks[prometheus]

# Error Tracking
pip install lazyhooks[sentry]

# Structured Logging
pip install lazyhooks[structlog]

# Everything
pip install lazyhooks[full]
```

## Features

### Prometheus Metrics

If `prometheus-client` is installed, the following metrics are automatically recorded:

- `lazyhooks_webhooks_sent_total` (Counter): Total webhooks sent. Labels: `destination`, `status` (`success`, `failure`).
- `lazyhooks_webhook_duration_seconds` (Histogram): Latency of webhook delivery. Labels: `destination`.
- `lazyhooks_webhook_retried_total` (Counter): Total retries scheduled. Labels: `destination`.

You must expose these metrics in your application (e.g., using `make_asgi_app` in FastAPI).

### Sentry Error Tracking

If `sentry-sdk` is installed and initialized in your app, failed webhooks are automatically reported to Sentry with the following context:

- **Tags**: `webhook.destination`, `webhook.id`
- **Extra**: `attempt`
- **Message**: "Webhook failed to {url}: {error}"

### Structured Logging

If `structlog` is installed, LazyHooks uses it to emit structured JSON logs (e.g., `webhook.sending`, `webhook.success`, `webhook.failed`). 

If not installed, it gracefully falls back to standard Python `logging`.

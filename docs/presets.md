# Configuration Presets

LazyHooks provides 4 core configuration presets to simplify setup.

## Usage

```python
from lazyhooks import presets

# Use a preset
sender = presets.production(
    signing_secret="my-secret",
    redis_url="redis://localhost:6379/0"
)

# Customize it
sender = presets.production(
    signing_secret="my-secret",
    redis_url="...",
    default_timeout=60  # Override timeout
)
```

## Available Presets

### 1. `development`
**Goal:** Speed and Debugging.
- Fails fast (1 retry).
- Very short timeout (5s).
- **Use for:** Local development, tests.

```python
sender = presets.development("secret")
```

### 2. `production`
**Goal:** Balanced Stability.
- Standard retries (5 attempts up to 1h).
- Standard timeout (10s).
- **Use for:** 90% of production apps.

```python
sender = presets.production("secret", "redis://...")
```

### 3. `strict`
**Goal:** Guaranteed Delivery.
- Aggressive retries (10 attempts up to 12h).
- Long timeout (30s).
- **Use for:** Payments, critical compliance.

```python
sender = presets.strict("secret", "redis://...")
```

### 4. `high_volume`
**Goal:** Throughput Protection.
- Fewer retries (3 attempts).
- Short timeout (5s).
- **Use for:** High scale (>10k/hr) non-critical events.

```python
sender = presets.high_volume("secret", "redis://...")
```

# Tutorial 3: Sending Your First Webhook

Now for the fun part—sending data!

We generally use **LazyHooks Presets** to configure things easily. Since we are just testing, we will use the `development` preset.

## The Script

Create a new file called `send_hook.py` and paste this code:

```python
from lazyhooks import presets

# 1. Setup the sender
# We use the 'development' preset. 
# WHY? Because configuring timeouts, retries, and storage by hand is boring.
# The preset gives us a perfect "Just Works" configuration for testing.
sender = presets.development(signing_secret="my-secret-key")

# 2. Define your data
data = {
    "message": "Hello World!",
    "status": "awesome"
}

# 3. Send it!
try:
    print(f" Sending webhook to https://httpbin.org/post...")
    sender.send_sync("https://httpbin.org/post", data)
    print(" Webhook sent successfully!")
    print("(The server received it and sent a 200 OK back)")
except Exception as e:
    print(f" Something went wrong: {e}")
```

## Run It

In your terminal:

```bash
python send_hook.py
```

You should see:
> Webhook sent successfully!

## What just happened?

1.  LazyHooks took your `data`.
2.  It calculated a secure **signature** using your `my-secret-key`.
3.  It sent the data to `https://httpbin.org/post` (a public testing echo service).
4.  It confirmed the server received it.

You just successfully communicated between systems!

[Next: Tutorial 4 - Receiving Webhooks ->](04_receiving_webhooks.md)

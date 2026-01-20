# Tutorial 4: Receiving Webhooks

Sending is half the battle. Now, let's capture a webhook sent to us.

## The Goal
We will build a tiny web server that listens for a webhook and checks if it's authentic.

We'll use **Flask**, a popular, simple Python web framework.
First, install Flask:
```bash
pip install flask
```

## The Receiver Script

Create `receive_hook.py`:

```python
from flask import Flask, request
from lazyhooks import verify_signature

app = Flask(__name__)

# This is our SECRET password. Must match the sender!
SECRET = "my-secret-key"

@app.route('/webhook', methods=['POST'])
def handle_webhook():
    # 1. Get the security headers
    signature = request.headers.get("X-Lh-Signature")
    timestamp = request.headers.get("X-Lh-Timestamp")
    
    # 2. Get the raw body bytes
    body = request.get_data()
    
    # 3. Verify it's actually from us!
    # Think of this like checking the wax seal on a letter.
    # If the seal is broken (signature doesn't match) or the letter is 
    # from last year (timestamp expired), we reject it.
    try:
        verify_signature(body, signature, SECRET, timestamp)
        print("\nSECURE WEBHOOK VERIFIED!")
        print(f"Data received: {request.json}")
        return "OK", 200
        
    except Exception as e:
        print(f"Hack attempt? Verification failed: {e}")
        return "Unauthorized", 401

if __name__ == '__main__':
    print("Listening on http://localhost:5000/webhook")
    app.run(port=5000)
```

## Testing It

1.  Run the receiver in one terminal:
    ```bash
    python receive_hook.py
    ```

2.  Modify your `send_hook.py` (from Tutorial 3) to point to your local server:
    ```python
    sender.send_sync("http://localhost:5000/webhook", data)
    ```

3.  Run the sender in a **second** terminal:
    ```bash
    python send_hook.py
    ```

**Result:**
Your receiver terminal should light up with:
> Secure webhook verified!
> Data received: {'message': 'Hello World!', 'status': 'awesome'}

## Congratulations!

You have built a complete, secure webhook pipeline. You are sending data, signing it for security, and verifying it on the other end.

**Where to go next?**
- [Configuration Presets](../presets.md) for production settings.
- [Error Handling](../error_handling.md) to make it robust.

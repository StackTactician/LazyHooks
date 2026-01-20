# Tutorial 5: Testing with Tunnels

So far, we've been comfortable on `localhost`. But what happens when you want **Stripe**, **GitHub**, or **Slack** to send you a webhook?

## The Problem

If you tell GitHub to send a webhook to `http://localhost:5000/webhook`, it will fail.
Why? Because `localhost` effectively means "this computer". GitHub tries to send it to *itself*, not to you.

Your computer is likely behind a wifi router and firewall. The outside internet cannot see it.

## The Solution: Tunnels

A **Tunnel** safely exposes a specific port on your computer to the public internet temporarily. It gives you a public URL (like `https://random-name.com`) that forwards traffic to your `localhost:5000`.

## Using ngrok

The industry standard tool for this is **ngrok**.

### 1. Install ngrok
Download it from [ngrok.com](https://ngrok.com/download) or install via package manager:
- macOS: `brew install ngrok/ngrok/ngrok`
- Windows: `choco install ngrok`

### 2. Start your Receiver
Run your script from Tutorial 4:
```bash
python receive_hook.py
# Listening on http://localhost:5000/webhook
```

### 3. Start the Tunnel
Open a **new terminal window** and run:
```bash
ngrok http 5000
```

You will see a screen like this:
```
Forwarding                    https://1234-56-78-90.ngrok-free.app -> http://localhost:5000
```

### 4. Test it
Copy that Forwarding URL (`https://1234-56-78-90.ngrok-free.app`) and use it!

- **In your Python Sender**:
  ```python
  sender.send_sync("https://1234-56-78-90.ngrok-free.app/webhook", data)
  ```

- **In GitHub/Stripe Settings**: Paste the ngrok URL as the "Payload URL".

## Troubleshooting 🔧

### "ERR_CONNECTION_REFUSED"
This usually means your Flask app isn't running.
- **Fix**: Check your other terminal. Is `python receive_hook.py` still running?

### "401 Unauthorized"
Your tunnel is working, but verification failed!
- **Fix**: Make sure `SECRET` in `receive_hook.py` matches the `signing_secret` exactly.

### "404 Not Found"
You might have the wrong URL path.
- **Fix**: Did you forget `/webhook` at the end? (e.g. `...ngrok-free.app/webhook`)

## Conclusion

Now you have a fully working webhook development environment:
1.  **LazyHooks** handles the security and logic.
2.  **Flask** handles the web server.
3.  **ngrok** connects you to the world.

Happy Hooking! moving data has never been this easy.

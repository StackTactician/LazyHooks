# Tutorial 1: What are Webhooks?

Welcome! If you've heard the word "Webhook" tossed around and nodded vaguely while secretly wondering what it meant—this guide is for you.

## The Simple Explanation

Imagine you are expecting an important package.

### ❌ The Old Way (Polling)
You walk to the mailbox every 5 minutes to check.
- **10:00**: Empty. (Wasted trip)
- **10:05**: Empty. (Wasted trip)
- **10:10**: Still empty. (Wasted trip)
- **10:15**: Package is there!

In programming, this is called **Polling**. It wastes time and resources checking repeatedly when nothing has changed.

### ✅ The Webhook Way
You give the courier your phone number and say, **"Text me when it arrives."**
- You sit on the couch and watch TV.
- **10:15**: *Ding!* "Package delivered."
- You go get it immediately.

In programming, a **Webhook** is that "Text Message." It allows one app to notify another app instantly when something happens.

---

## How it Works Technically

Let's use a real-world example: **You run an online T-Shirt shop.**

1.  **Event Happens**: A customer, Alice, buys a shirt for $25 on your **Stripe** checkout page.
2.  **Notification Sent**: Stripe looks at your settings. You told it: *"When a payment succeeds, tell me at `https://myshop.com/hooks/stripe`"*.
3.  **Data Delivered**: Stripe sends a `POST` request (like a digital letter) to your URL with the details:

```json
{
  "event_type": "payment_succeeded",
  "amount": 25.00,
  "currency": "USD",
  "customer_email": "alice@example.com",
  "receipt_id": "rcpt_12345"
}
```

4.  **Action Taken**: Your shop receives this, verifies it's really from Stripe, and instantly emails Alice her receipt.

## Common Use Cases

You interact with webhooks every day without knowing it:

-   **Payments**: PayPal/Stripe notifying a store that money arrived.
-   **Chat Bots**: Slack/Discord notifying a bot that a user typed `!help`.
-   **CI/CD**: GitHub notifying a build server (like Jenkins) that new code was pushed.
-   **SMS**: Twilio notifying your app that a user replied "STOP".

## Why use them?

## Why use them?

- **Speed**: You know *immediately* when things happen.
- **Efficiency**: No more wasting energy checking valid "empty mailboxes."
- **Simplicity**: It's just simple web messages flying back and forth.

[Next: Tutorial 2 - Installation ->](02_installation.md)

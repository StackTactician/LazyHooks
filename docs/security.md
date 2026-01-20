# Security

LazyHooks prioritizes security by design.

## HMAC Signatures
All webhooks are signed using HMAC-SHA256. The signature is computed on `timestamp.body` to prevent tampering.

Headers:
- `X-Lh-Timestamp`: Unix timestamp
- `X-Lh-Signature`: `v1=...`

## Replay Protection
To prevent replay attacks, `WebhookReceiver` (and `verify_signature`) enforce a strict timestamp window (default 5 minutes). Requests older than this are rejected, even if the signature is valid.

## Best Practices
1. **Keep Secrets Safe**: Never commit your signing secret to git. Use environment variables.
2. **Rotate Secrets**: If compromised, change your secret and update both sender and receiver.
3. **Use HTTPS**: Signatures don't encrypt data, they only verify authenticity. Always use HTTPS.

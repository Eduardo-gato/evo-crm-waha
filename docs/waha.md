# WAHA Provider

The CRM integrates WAHA as an independent WhatsApp provider. Evolution API and Evolution Go keep their existing routes, payloads, sessions and webhook handlers.

## Local Setup

Set the following values in `.env`:

```text
WAHA_ENABLED=true
WAHA_API_URL=http://waha:3000
WAHA_API_KEY=change-me
WAHA_WEBHOOK_HMAC_KEY=change-me-too
WAHA_WEBHOOK_BASE_URL=https://crm.example.com/webhooks/whatsapp/waha
WAHA_DEFAULT_ENGINE=GOWS
```

Start the optional WAHA service with:

```bash
docker compose -f docker-compose.yml -f docker-compose.waha.yml --profile waha up -d waha
```

The WAHA API should remain on the internal Docker network. Only the CRM webhook URL needs to be reachable by the WAHA container.

## Channel Flow

1. Select `WhatsApp` and provider `WAHA` in the CRM.
2. Test the WAHA connection (URL + API key only when no global WAHA config exists).
3. Create the channel. The CRM creates a dedicated WAHA session (`evo-<channel>-<hex>`; a custom name can be set under Advanced) and registers the webhook. No phone number is requested: WAHA is paired before the number is known.
4. Open the channel configuration and request the QR Code.
5. Scan the QR Code from WhatsApp Linked Devices.
6. The `session.status` webhook changes the CRM channel to connected after `WORKING`, and the CRM reads the paired number from `GET /api/sessions/{session}/me`.

The default engine is `GOWS`; the channel form also allows `WEBJS`, `NOWEB` and `WPP`. Each channel gets its own session, so multiple WhatsApp accounts can be connected at once on WAHA Plus (recent free builds also accept multiple sessions).

## Security

- `WAHA_API_KEY` is sent only from the CRM backend to WAHA.
- `X-Webhook-Hmac` is validated against the raw request body with SHA-512.
- Unknown WAHA sessions are rejected before enqueueing a job.
- API keys and HMAC keys are removed from Inbox API responses.
- Media is downloaded server-side with the WAHA API key and stored in ActiveStorage.
- Do not expose WAHA publicly without authentication and HTTPS.

## WhatsApp Risk

WAHA uses WhatsApp Web automation and is not the official Meta WhatsApp Cloud API. It can be blocked by WhatsApp and should not be used as the only provider for critical or regulated communication. Do not connect the same WhatsApp account to WAHA and Evolution Go at the same time.

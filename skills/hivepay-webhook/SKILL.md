---
name: hivepay-webhook
description: Set up HivePay webhook handling and signature verification. Use when the user wants to receive payment notifications, verify webhook signatures, configure webhook URLs, or handle payment status change events. Triggers on mentions of webhook, signature verification, or payment notifications.
---

# HivePay Webhook Skill

You are helping the user set up webhook handling for HivePay — a payment gateway for the Hive blockchain. Webhooks notify your server when payment statuses change.

**Prerequisites:** The user should already have `@hivepay/client` installed and a client instance configured with `webhookSecret`. If not, guide them to run the `hivepay-setup` skill first.

## Step 1: Detect the user's project environment

Before writing any code, investigate the user's project to understand their server framework (Nuxt, Next.js, Express, SvelteKit, etc.), TypeScript vs JavaScript, and existing patterns. Adapt all code to match.

## Step 2: Generate webhook handler

### Webhook payload format

```json
{
  "type": "payment.status_changed",
  "data": {
    "id": "payment_id",
    "merchantId": "merchant_id",
    "status": "completed",
    "providerId": "hive",
    "internalId": "provider_reference"
  }
}
```

### Headers sent with each webhook

- `X-HivePay-Signature` — HMAC-SHA256 hex signature
- `X-HivePay-Timestamp` — Unix timestamp in milliseconds

### Signature algorithm

`HMAC_SHA256(webhookSecret, "${timestamp}.${JSON.stringify(payload)}")`

**CRITICAL:** Always verify the webhook signature before processing. The SDK provides `hivepay.verifyWebhook()` for this.

### Framework examples

**Express:**
```typescript
app.post('/webhooks/hivepay', express.raw({ type: 'application/json' }), async (req, res) => {
  const result = await hivepay.verifyWebhook({
    payload: req.body.toString(),
    signature: req.header('X-HivePay-Signature')!,
    timestamp: req.header('X-HivePay-Timestamp')!,
  });

  if (!result.valid) {
    return res.status(401).json({ error: result.error });
  }

  const { event } = result;

  switch (event.data.status) {
    case 'completed':
      // Fulfill the order
      break;
    case 'failed':
      // Handle failure
      break;
    case 'user_cancelled':
    case 'system_cancelled':
      // Handle cancellation
      break;
  }

  res.json({ received: true });
});
```

**Next.js App Router:**
```typescript
import { NextResponse } from 'next/server';
import { hivepay } from '@/lib/hivepay';

export async function POST(request: Request) {
  const result = await hivepay.verifyWebhook({
    payload: await request.text(),
    signature: request.headers.get('X-HivePay-Signature')!,
    timestamp: request.headers.get('X-HivePay-Timestamp')!,
  });

  if (!result.valid) {
    return NextResponse.json({ error: result.error }, { status: 401 });
  }

  const { event } = result;

  switch (event.data.status) {
    case 'completed':
      // Fulfill the order
      break;
    case 'failed':
      // Handle failure
      break;
  }

  return NextResponse.json({ received: true });
}
```

**Nuxt/Nitro:**
```typescript
import { useHivePay } from '@/server/utils/hivepay';

export default defineEventHandler(async (event) => {
  const hivepay = useHivePay();
  const body = await readRawBody(event);

  const result = await hivepay.verifyWebhook({
    payload: body!,
    signature: getHeader(event, 'x-hivepay-signature')!,
    timestamp: getHeader(event, 'x-hivepay-timestamp')!,
  });

  if (!result.valid) {
    throw createError({ statusCode: 401, statusMessage: result.error });
  }

  const { event: whEvent } = result;

  switch (whEvent.data.status) {
    case 'completed':
      // Fulfill the order
      break;
    case 'failed':
      // Handle failure
      break;
  }

  return { received: true };
});
```

**Hono:**
```typescript
app.post('/webhooks/hivepay', async (c) => {
  const result = await hivepay.verifyWebhook({
    payload: await c.req.text(),
    signature: c.req.header('X-HivePay-Signature')!,
    timestamp: c.req.header('X-HivePay-Timestamp')!,
  });

  if (!result.valid) {
    return c.json({ error: result.error }, 401);
  }

  // Handle event...
  return c.json({ received: true });
});
```

**SvelteKit:**
```typescript
import { hivepay } from '$lib/server/hivepay';

export async function POST({ request }) {
  const result = await hivepay.verifyWebhook({
    payload: await request.text(),
    signature: request.headers.get('X-HivePay-Signature')!,
    timestamp: request.headers.get('X-HivePay-Timestamp')!,
  });

  if (!result.valid) {
    return new Response(JSON.stringify({ error: result.error }), { status: 401 });
  }

  // Handle event...
  return new Response(JSON.stringify({ received: true }));
}
```

## Security best practices

1. Always verify signatures — never process unverified webhooks
2. Check timestamps — reject webhooks older than 5 minutes (default in SDK)
3. Use HTTPS in production
4. Respond within 5 seconds — do heavy processing asynchronously
5. Implement idempotency — you may receive duplicate webhooks

## Configure webhook URL

```typescript
await hivepay.merchants.update('merchant_id', {
  webhookUrl: 'https://yoursite.com/webhooks/hivepay',
});
```

## Regenerate webhook secret

```typescript
const result = await hivepay.merchants.regenerateWebhookSecret('merchant_id');
// result.webhookSecret — new secret (shown only once; old one is immediately invalidated)
```

## Key Resources

- Full LLM documentation: https://docs.hivepay.me/llms-full.txt
- Webhook format details: https://docs.hivepay.me/llms-full.txt

---
name: hivepay-payment
description: Create and track HivePay payment sessions. Use when the user wants to create payments, check payment status, list transactions, poll for completion, or build a checkout flow with HivePay. Triggers on mentions of payment creation, checkout, payment status, or transaction listing.
---

# HivePay Payment Skill

You are helping the user create and manage HivePay payment sessions. HivePay provides a Stripe-like REST API and TypeScript SDK (`@hivepay/client`) for accepting HIVE and HBD cryptocurrency payments.

**Prerequisites:** The user should already have `@hivepay/client` installed and a client instance configured. If not, guide them to run the `hivepay-setup` skill first.

## Step 1: Detect the user's project environment

Before writing any code, investigate the user's project to understand their server framework (Nuxt, Next.js, Express, SvelteKit, etc.), TypeScript vs JavaScript, and existing patterns for imports, error handling, and HTTP calls. Adapt all code to match.

## Step 2: Generate payment code

### Create a payment endpoint

The endpoint should:
1. Accept `amount` (string, in satoshis — "10500" = 10.500 HIVE/HBD), `currency` ("HIVE" or "HBD"), `description`, and optional `redirectUrl`/`cancelUrl`/`metadata`
2. Call `hivepay.payments.create()`
3. Return the `checkoutUrl` for the client to redirect to

**Example — Express:**
```typescript
app.post('/api/payments/create', async (req, res) => {
  try {
    const payment = await hivepay.payments.create({
      amount: req.body.amount,
      currency: req.body.currency,
      description: req.body.description,
      redirectUrl: req.body.redirectUrl,
      cancelUrl: req.body.cancelUrl,
      metadata: req.body.metadata,
    });
    res.json({ checkoutUrl: payment.checkoutUrl, id: payment.id });
  } catch (error) {
    if (isHivePayError(error)) {
      res.status(error.statusCode ?? 500).json({ error: error.message });
    } else {
      res.status(500).json({ error: 'Payment creation failed' });
    }
  }
});
```

**Example — Next.js App Router:**
```typescript
import { NextResponse } from 'next/server';
import { hivepay } from '@/lib/hivepay';
import { isHivePayError } from '@hivepay/client';

export async function POST(request: Request) {
  const body = await request.json();

  try {
    const payment = await hivepay.payments.create({
      amount: body.amount,
      currency: body.currency,
      description: body.description,
      redirectUrl: body.redirectUrl,
      cancelUrl: body.cancelUrl,
      metadata: body.metadata,
    });
    return NextResponse.json({ checkoutUrl: payment.checkoutUrl, id: payment.id });
  } catch (error) {
    if (isHivePayError(error)) {
      return NextResponse.json({ error: error.message }, { status: error.statusCode ?? 500 });
    }
    return NextResponse.json({ error: 'Payment creation failed' }, { status: 500 });
  }
}
```

**Example — Nuxt/Nitro:**
```typescript
import { useHivePay } from '@/server/utils/hivepay';
import { isHivePayError } from '@hivepay/client';

export default defineEventHandler(async (event) => {
  const body = await readBody(event);
  const hivepay = useHivePay();

  try {
    const payment = await hivepay.payments.create({
      amount: body.amount,
      currency: body.currency,
      description: body.description,
      redirectUrl: body.redirectUrl,
      cancelUrl: body.cancelUrl,
      metadata: body.metadata,
    });
    return { checkoutUrl: payment.checkoutUrl, id: payment.id };
  } catch (error) {
    if (isHivePayError(error)) {
      throw createError({ statusCode: error.statusCode ?? 500, statusMessage: error.message });
    }
    throw createError({ statusCode: 500, statusMessage: 'Payment creation failed' });
  }
});
```

### Amount format

Amounts are strings in satoshis (smallest unit). Both HIVE and HBD use 3 decimal precision:

| Input | Actual Amount |
|-------|---------------|
| `"1000"` | 1.000 HIVE/HBD |
| `"10500"` | 10.500 HIVE/HBD |
| `"100000"` | 100.000 HIVE/HBD |

### Check payment status

```typescript
const status = await hivepay.payments.getStatus('payment_id');
// Returns: 'pending' | 'processing' | 'completed' | 'failed' | 'user_cancelled' | 'system_cancelled'
```

### Poll until complete

```typescript
const finalStatus = await hivepay.payments.waitFor('payment_id', {
  timeout: 300000,  // 5 minutes
  interval: 3000,   // check every 3 seconds
});
```

### List payments (paginated)

```typescript
const result = await hivepay.payments.list({ page: 1, limit: 20 });
// result.data — Payment[]
// result.pagination — { page, limit, total, totalPages }
```

### Iterate all payments

```typescript
for await (const payment of hivepay.payments) {
  console.log(payment.id, payment.status);
}
```

### Payment lifecycle

```
pending → processing → completed | failed | user_cancelled | system_cancelled
```

## Error Handling

```typescript
import { isHivePayError } from '@hivepay/client';

try {
  await hivepay.payments.create({ ... });
} catch (error) {
  if (isHivePayError(error)) {
    error.isAuthError();     // 401
    error.isValidation();    // 400
    error.isRateLimited();   // 429
    error.isServerError();   // 5xx
    error.isNetworkError();  // network/timeout
  }
}
```

## Key Resources

- Full LLM documentation: https://docs.hivepay.me/llms-full.txt
- OpenAPI specification: https://hivepay.me/openapi.json

---
name: hivepay-x402
description: Set up x402 HTTP-native micropayments with HivePay. Use when the user wants to implement HTTP 402 Payment Required flows, build AI agent payment clients, verify or settle x402 transactions, or enable machine-to-machine payments on Hive. Triggers on mentions of x402, HTTP 402, micropayments, AI agent payments, or HiveX402Client.
---

# HivePay x402 Skill

You are helping the user work with HivePay's x402 facilitator for HTTP 402 Payment Required flows on the Hive blockchain. This enables machine-to-machine micropayments, ideal for AI agents and automated services.

**Prerequisites:** The user should already have `@hivepay/client` installed and a client instance configured. If not, guide them to run the `hivepay-setup` skill first.

## How x402 works

When an x402-aware client (AI agent, `HiveX402Client`) hits a checkout URL, HivePay returns `402 Payment Required` with payment requirements. The client signs a Hive transaction with two transfers (net to merchant + fee to HivePay) and retries with the `x-payment` header. HivePay verifies, broadcasts, and marks the session completed.

Checkout URLs returned by `payments.create()` automatically work for both browsers and x402 clients — no additional merchant setup needed.

## Toggle x402 support

```typescript
// Enabled by default for all merchants
await hivepay.merchants.update('merchant_id', { x402Enabled: false }); // disable
await hivepay.merchants.update('merchant_id', { x402Enabled: true });  // re-enable

// Check status
const merchant = await hivepay.merchants.getCurrent();
console.log(merchant.x402Enabled); // true
```

## Verify a payment payload (without broadcasting)

```typescript
const result = await hivepay.payments.x402Verify('session_id', {
  x402Version: 1,
  scheme: 'exact',
  network: 'hive:mainnet',
  payload: {
    signedTransaction: tx,  // HF26-format signed Hive transaction
    nonce: 'unique-nonce',  // Unique per payment (replay protection)
  },
});

if (result.isValid) {
  console.log('Valid payment from:', result.payer);
} else {
  console.log('Invalid:', result.invalidReason);
}
```

## Settle a payment (verify + broadcast + complete session)

```typescript
const result = await hivepay.payments.x402Settle('session_id', {
  x402Version: 1,
  scheme: 'exact',
  network: 'hive:mainnet',
  payload: {
    signedTransaction: tx,
    nonce: 'unique-nonce',
  },
});

if (result.success) {
  console.log('TX:', result.txId, 'Payer:', result.payer);
} else {
  console.log('Failed:', result.errorReason);
}
```

## Transaction structure

The signed transaction must contain exactly two transfer operations:

| Transfer | Recipient | Amount |
|----------|-----------|--------|
| Net payment | Merchant's Hive account (`payTo`) | `netAmount` from 402 response |
| Fee | HivePay fee account (`extra.feeAccount`) | `feeAmount` from 402 response |

Both transfers must be from the same sender and in the same currency as the session.

## Client-side (AI agents)

Install `@hiveio/x402` and use `HiveX402Client` to pay for checkout URLs automatically:

```typescript
import { HiveX402Client } from '@hiveio/x402/client';

const client = new HiveX402Client({
  account: 'alice',
  activeKey: '5K...',   // Hive active private key (WIF)
  maxPayment: 15.0,     // Max HBD per request (safety limit)
});

const response = await client.fetch('https://hivepay.me/for/order-abc123');
const data = await response.json();
// { success: true, txId: "abc123...", payer: "alice", sessionId: "...", status: "completed" }
```

## Key Resources

- Full LLM documentation: https://docs.hivepay.me/llms-full.txt
- OpenAPI specification: https://hivepay.me/openapi.json

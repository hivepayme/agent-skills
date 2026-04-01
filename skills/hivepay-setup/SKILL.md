---
name: hivepay-setup
description: Install and configure the HivePay SDK (@hivepay/client) in a project. Use when the user wants to add HivePay, set up API keys, create a client instance, or register a new merchant account. Triggers on mentions of HivePay setup, install, configure, or API key.
---

# HivePay Setup Skill

You are helping the user install and configure HivePay — a payment gateway for the Hive blockchain — in their project. HivePay provides a Stripe-like REST API and TypeScript SDK (`@hivepay/client`) for accepting HIVE and HBD cryptocurrency payments.

## Step 1: Detect the user's project environment

Before writing any code, you MUST investigate the user's project to understand:

1. **Package manager**: Check for `pnpm-lock.yaml`, `yarn.lock`, `bun.lock`, `package-lock.json`, or `deno.json`
2. **Server framework**: Look for indicators of:
   - **Nuxt/Nitro**: `nuxt.config.ts`, `server/api/` directory
   - **Next.js**: `next.config.*`, `app/api/` or `pages/api/`
   - **Express**: `express` in dependencies
   - **Fastify**: `fastify` in dependencies
   - **NestJS**: `@nestjs/core` in dependencies
   - **SvelteKit**: `svelte.config.*`, `src/routes/+server.ts`
   - **Remix**: `@remix-run/node` in dependencies
   - **Astro**: `astro.config.*`
   - **Deno/Fresh**: `deno.json`, `fresh.config.ts`
   - **Bun**: `Bun.serve` usage
   - **No framework**: Plain Node.js with `http`/`https` module
3. **TypeScript or JavaScript**: Check for `tsconfig.json`
4. **Environment variable patterns**: Check for `.env`, `.env.example`, `.env.local`, or framework-specific env handling (e.g., `process.env`, `import.meta.env`, `useRuntimeConfig()`)
5. **Existing HTTP client patterns**: How the project makes outbound HTTP calls (native `fetch`, `axios`, `got`, `ofetch`, etc.)
6. **Module system**: ESM (`"type": "module"`) or CommonJS

Adapt all code examples to match the detected patterns. Use the project's existing conventions for imports, error handling, environment variables, and HTTP calls.

## Step 2: Install and configure

1. Install the SDK using the detected package manager:
   - pnpm: `pnpm add @hivepay/client`
   - yarn: `yarn add @hivepay/client`
   - npm: `npm install @hivepay/client`
   - bun: `bun add @hivepay/client`

2. Add environment variables to the project's env file (`.env`, `.env.local`, etc.):
   ```
   HIVEPAY_API_KEY=sk_live_your_api_key_here
   HIVEPAY_WEBHOOK_SECRET=whsec_your_webhook_secret_here
   ```

3. Create a shared HivePay client instance following the project's patterns. Adapt to the framework:

   **Generic / Express / Fastify / Hono:**
   ```typescript
   import { HivePay } from '@hivepay/client';

   export const hivepay = new HivePay({
     apiKey: process.env.HIVEPAY_API_KEY,
     webhookSecret: process.env.HIVEPAY_WEBHOOK_SECRET,
   });
   ```

   **Nuxt/Nitro:**
   ```typescript
   import { HivePay } from '@hivepay/client';

   let _client: HivePay | undefined;

   export const useHivePay = () => {
     if (!_client) {
       const config = useRuntimeConfig();
       _client = new HivePay({
         apiKey: config.hivepayApiKey,
         webhookSecret: config.hivepayWebhookSecret,
       });
     }
     return _client;
   };
   ```
   Also add to `nuxt.config.ts` runtimeConfig:
   ```typescript
   runtimeConfig: {
     hivepayApiKey: process.env.HIVEPAY_API_KEY,
     hivepayWebhookSecret: process.env.HIVEPAY_WEBHOOK_SECRET,
   }
   ```

   **Next.js (App Router):**
   ```typescript
   import { HivePay } from '@hivepay/client';

   export const hivepay = new HivePay({
     apiKey: process.env.HIVEPAY_API_KEY!,
     webhookSecret: process.env.HIVEPAY_WEBHOOK_SECRET!,
   });
   ```

   **SvelteKit:**
   ```typescript
   import { HivePay } from '@hivepay/client';
   import { HIVEPAY_API_KEY, HIVEPAY_WEBHOOK_SECRET } from '$env/static/private';

   export const hivepay = new HivePay({
     apiKey: HIVEPAY_API_KEY,
     webhookSecret: HIVEPAY_WEBHOOK_SECRET,
   });
   ```

4. If the user doesn't have a HivePay account yet, show the registration flow:
   ```typescript
   import { HivePay } from '@hivepay/client';

   const publicClient = new HivePay();
   const result = await publicClient.merchants.register({
     name: 'My Store',
     hiveAccount: 'your_hive_account',
   });

   // SAVE THESE — they are shown only once:
   // API Key: result.apiKey
   // Webhook Secret: result.webhookSecret
   // Merchant ID: result.merchant.id
   ```

## SDK Reference

- **Authentication**: Header `Authorization: Bearer sk_live_xxx` (or `X-API-Key: sk_live_xxx`)
- **Key format**: `sk_live_` + 48 hex chars (production) or `sk_test_` + 48 hex chars (development)
- **No auth required** for merchant registration
- **Supported currencies**: HIVE (3 decimals), HBD (3 decimals)
- **Requirements**: Node.js 18+, zero dependencies, ESM module, TypeScript included

## Error Handling

```typescript
import { isHivePayError } from '@hivepay/client';

try {
  await hivepay.payments.create({ ... });
} catch (error) {
  if (isHivePayError(error)) {
    error.isAuthError();     // 401
    error.isForbidden();     // 403
    error.isNotFound();      // 404
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
- npm package: https://www.npmjs.com/package/@hivepay/client
- Dashboard: https://dashboard.hivepay.me

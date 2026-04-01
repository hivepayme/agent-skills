---
name: hivepay-fees
description: Calculate HivePay payment fees and format amounts. Use when the user wants to compute fee splits, convert between satoshis and display amounts, understand the fee structure, or get the current fee rate. Triggers on mentions of fee calculation, amount formatting, satoshis, or payment splitting.
---

# HivePay Fees Skill

You are helping the user work with HivePay fee calculations and amount formatting. HivePay uses satoshi-based amounts (strings) with 3-decimal precision for both HIVE and HBD.

**Prerequisites:** The user should already have `@hivepay/client` installed. If not, guide them to run the `hivepay-setup` skill first.

## Fee calculations

```typescript
import { getSplitAmount, getSplitAmountFromFormatted, formatSatoshis } from '@hivepay/client';

// From satoshis (bigint)
const { fee, net } = getSplitAmount(10500n, 1.5);
// fee = 157n, net = 10343n

// From formatted decimal string
const { fee, net } = getSplitAmountFromFormatted('10.500', 1.5);
```

## Format amounts for display

```typescript
import { formatSatoshis } from '@hivepay/client';

formatSatoshis('10500');     // "10.500"
formatSatoshis('10500', 2);  // "10.50"
```

## Get current fee rate from API

```typescript
const feePercent = await hivepay.payments.getFeeRate();
```

## Fee formula

`fee = floor(amount * feePercent / 100)`, minimum 1 satoshi. `net = amount - fee`.

## Amount format reference

Amounts are strings in satoshis (smallest unit). Both HIVE and HBD use 3 decimal precision:

| Input | Actual Amount |
|-------|---------------|
| `"1000"` | 1.000 HIVE/HBD |
| `"10500"` | 10.500 HIVE/HBD |
| `"100000"` | 100.000 HIVE/HBD |

## Key Resources

- Full LLM documentation: https://docs.hivepay.me/llms-full.txt
- npm package: https://www.npmjs.com/package/@hivepay/client

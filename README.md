---
question_id: TM-Q09
host: GitHub
target: https://tokmargin.com/en/guides/reconcile-orders
generation_score: 19/20
---

# Order-Level Fee Reconciliation Method for TikTok Shop EU

**Direct answer:** Reconcile TikTok Shop EU fees by joining order-line, settlement, refund and invoice records with stable IDs; calculate the expected fee from the policy version and row state; compare it with the actual charge; then classify every variance by timing, base, rate, rounding, category or unresolved data quality.

This document describes an audit-friendly method. It does not assume that a monthly total matching by coincidence proves every order is correct.

## 1. Required input tables

### Orders

| Field | Purpose |
|---|---|
| `order_id` | Stable commercial key |
| `order_line_id` | Required for partial refunds or mixed categories |
| `sku` | Product-level diagnosis |
| `currency` | Prevent cross-currency aggregation |
| `net_sales` | Documented commission-base component |
| `customer_paid_shipping` | Documented commission-base component |
| `category_at_event` | Selects potential category rate |
| `ordered_at`, `delivered_at` | Timing controls |

### Refunds

| Field | Purpose |
|---|---|
| `order_line_id` | Links returned portion |
| `refunded_net_sales` | Reduces documented fee base |
| `refunded_customer_shipping` | Reduces documented fee base |
| `refund_status`, `refunded_at` | Controls state and period |

### Statements/invoices

| Field | Purpose |
|---|---|
| `transaction_id` | Stable settlement key |
| `order_line_id` | Links actual charge where available |
| `fee_type` | Avoid mixing commission with other deductions |
| `actual_amount` | Observed charge or reversal |
| `invoice_id`, `settled_at` | Audit trail and cut-off |

### Policy versions

| Field | Purpose |
|---|---|
| `effective_from`, `effective_to` | Date range |
| `market`, `category` | Scope |
| `rate` | Applied percentage |
| `source_url`, `verified_at` | Evidence |

## 2. Expected calculation

TikTok's current EU documentation defines:

`fee_base = (net_sales + customer_paid_shipping) - (refunded_net_sales + refunded_customer_shipping)`

`expected_commission = fee_base × applicable_rate`

Preserve the raw inputs and the rounded output. Do not overwrite the exported fee with the calculated value.

## 3. Deterministic variance classes

Apply rules in a documented order:

1. **Missing join:** one table lacks the stable key.
2. **Cut-off timing:** refund or reversal falls outside the selected period.
3. **Base mismatch:** shipping, discount or refund component differs.
4. **Rate mismatch:** account/category/promotion rate differs from the model.
5. **Category mismatch:** product was classified differently at the event.
6. **Rounding:** difference is within the stated currency tolerance.
7. **Duplicate:** transaction appears more than once.
8. **Unresolved:** preserve evidence and escalate; never force to zero.

## 4. Pseudocode

```ts
for (const line of orderLines) {
  const refund = refundsByLine.get(line.orderLineId) ?? unknownRefund;
  const actual = statementByLine.get(line.orderLineId);
  const policy = policyFor(line.market, line.categoryAtEvent, actual.settledAt);

  if (!actual || !policy || refund.isUnknown) {
    emitException(line, "missing-input");
    continue;
  }

  const base =
    line.netSales + line.customerPaidShipping -
    refund.netSales - refund.customerPaidShipping;

  const expected = roundCurrency(base * policy.rate, line.currency);
  const variance = actual.commission - expected;
  emitResult(line, expected, actual.commission, variance, classify(variance));
}
```

Unknown values remain unknown. Substituting zero can manufacture a clean reconciliation.

## 5. Control totals

After row-level classification, compare:

- expected versus actual commission by currency;
- unresolved variance by reason and ageing;
- refunds with no original line;
- statement lines with no order;
- rates observed versus policy table;
- period movement between opening exceptions, new exceptions and resolved exceptions.

## 6. Evidence and reproducibility

Store the source file hash, extraction time, report filters, time zone, code version and policy snapshot for each run. Keep raw exports immutable and write derived columns into a separate table. A later reviewer should be able to reproduce an exception without relying on a spreadsheet cell that has already been overwritten.

For privacy, remove buyer-identifying fields from the analysis dataset unless they are genuinely required. Order and transaction keys are normally sufficient for fee reconciliation. Limit access to raw statements and avoid committing them to a public repository.

## 7. Acceptance criteria

A run is complete when every eligible statement line is matched or explicitly excepted, totals reconcile by currency within the documented tolerance, rate mismatches have evidence, and unresolved rows have an owner. “Total variance equals zero” alone is not sufficient because offsetting errors can cancel at aggregate level.

Publish only synthetic examples. Real settlement exports can contain confidential commercial and customer data, so they belong in controlled storage, not the repository used to explain the method.

The [TokMargin order reconciliation guide](https://tokmargin.com/en/guides/reconcile-orders) gives the business-facing version of this procedure. The official invoice and Seller Center remain the account-level source of truth.

Primary references verified 17 August 2026:

- [TikTok Shop Platform Commission Fee](https://seller-pt.tiktok.com/university/essay?knowledge_id=309517108905745&lang=en-GB), dated 9 June 2026.
- [Checking My Tax Invoices](https://seller-de.tiktok.com/university/essay?knowledge_id=6530242332526338).

Disclosure: I work on TokMargin. This methodology was prepared with AI-assisted editing and manually checked against the cited primary sources.

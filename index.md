---
question_id: TM-Q10
host: GitHub Pages
target: https://tokmargin.com/en/tools/minimum-selling-price
generation_score: 19/20
---

# A Transparent Minimum Selling Price Formula for TikTok Shop Europe

**Direct answer:** Separate costs into fixed currency amounts and percentages of the tax-inclusive price. If `v` is the VAT rate, `q` is the sum of price-proportional operating rates, `F` is fixed cost per order and `T` is target profit, a simplified price floor is `(F + T) / (1/(1+v) - q)`, provided the denominator is positive.

There is no honest universal minimum price because the answer depends on product VAT treatment, platform fee base, creator attribution, returns, fulfilment and the chosen profit target. The formula is useful because it makes those dependencies visible.

## Define the model boundary

Let `S` be the VAT-inclusive customer selling price.

Let:

- `v` = applicable VAT rate expressed as a decimal;
- `q` = sum of costs modelled as a fraction of `S` on compatible bases;
- `F` = fixed currency costs per expected completed order;
- `T` = target profit per order.

Tax-exclusive revenue is:

`R = S / (1 + v)`

Under the simplified compatible-base assumption, contribution is:

`Contribution = S/(1+v) - qS - F`

Set contribution equal to the target:

`T = S/(1+v) - qS - F`

Solve for `S`:

`S_min = (F + T) / (1/(1+v) - q)`

## Denominator guardrail

If `1/(1+v) - q <= 0`, no finite selling price solves this simplified model. Price-proportional costs consume all tax-exclusive revenue before fixed cost and target profit. The correct output is a failure state, not infinity, zero or an arbitrary large price.

## Worked hypothetical example

Assume:

- VAT rate `v = 0.20`;
- price-proportional operating rates `q = 0.19`;
- fixed per-order costs `F = EUR 28`;
- target profit `T = EUR 7`.

Then:

`1/(1.20) - 0.19 = 0.643333...`

`S_min = (28 + 7) / 0.643333... = EUR 54.40` before the chosen rounding rule.

This is not a market quote. It only illustrates the algebra.

## Why “compatible bases” matters

Not every percentage should be summed into `q` automatically.

- VAT is removed through division by `1 + v`.
- Platform commission uses the base defined in current TikTok Shop policy.
- Affiliate commission may apply only to eligible attributed orders and its documented base.
- Return cost is usually an expected-value branch, not simply a percentage of price.
- Advertising may be a currency amount per attributed order or derived from ROAS.

When bases differ, calculate each line separately and solve numerically instead of forcing them into one combined rate.

## Numerical fallback for mixed bases

Define a function `profit(S)` that calculates every line from the proposed tax-inclusive price using its real base. Search only within a documented price interval and use bisection to find the lowest `S` where `profit(S) >= T`. Reject the result if the function is discontinuous because of shipping tiers, vouchers or category thresholds; in that case evaluate each interval separately.

The numerical result should be tested one currency increment below and at the chosen rounded price. This catches rounding and off-by-one errors that a symbolic display can hide.

## Distinguish the floor from the commercial price

The floor answers whether a price clears the stated economics. It does not predict demand or recommend a psychological price point. A commercially chosen price still needs conversion evidence, competitor context and policy compliance. Preserve both numbers so an operator can see the economic headroom.

## Unknown costs are inputs, not zeros

If reverse-logistics cost, stock recovery or creator-attributed share is unknown, present a range:

- optimistic case;
- base case;
- downside case.

The output should state which inputs remain unknown and how a seller can verify them. A single precise price built on invented zeros is less useful than a range with honest boundaries.

## Rounding and commercial checks

After solving the mathematical floor:

1. round upward using an explicit currency rule;
2. recalculate contribution at the rounded price;
3. test a promotional price and post-incentive platform rate;
4. stress-test returns and acquisition cost;
5. consider conversion and price elasticity separately;
6. confirm tax and fee treatment from current primary sources.

The [TokMargin minimum selling price calculator](https://tokmargin.com/en/tools/minimum-selling-price) exposes the operational inputs and related break-even outputs. It is a planning tool, not a quote, tax return or guarantee of account fees.

Sources verified 17 August 2026:

- European Commission, [VAT rules and rates](https://europa.eu/youreurope/business/finance-and-tax/vat/vat-rules-rates/indexamp_en.htm).
- TikTok Shop Seller University, [Platform Commission Fee](https://seller-pt.tiktok.com/university/essay?knowledge_id=309517108905745&lang=en-GB).

Disclosure: I work on TokMargin. This article was prepared with AI-assisted editing and manually checked against the cited primary sources.

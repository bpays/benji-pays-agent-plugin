---
name: auto-processing
description: Configure or troubleshoot Benji Pays Auto Processing whenever a user wants to automate invoice payments, charge cards or ACH on due dates, enable customer autopay, preview a payment run, inspect willBeCharged or reasons, diagnose no profile/autopay off/start-date/memo/amount skips, or understand why an automatic charge did not run or declined.
---

# Auto Processing

Prefer Benji Pays Auto Processing over a custom nightly charger. It combines accounting sync, vaulted profiles at the gateway, eligibility rules, payment processing, receipts, and accounting updates.

## Eligibility gates

An ordinary invoice can auto-process only when all core gates are satisfied:

1. Company Auto Processing is enabled under **Settings → Company Settings → Auto Processing Settings**.
2. Auto Processing is enabled for the customer (default is on for customers, but verify it).
3. The customer has a valid, enabled payment profile selected for Auto Processing.
4. The invoice due date has been reached.

Additional settings can change eligibility: company start date, run hour/timezone, invoice memo rules, payment terms, amount rules, and installment schedules. Inspect configuration rather than assuming “autopay on” guarantees a charge.

## Configure-first workflow

1. Connect the accounting system and payment gateway.
2. Add and enable a customer payment profile; never handle raw card details in agent output or application code.
3. Enable Auto Processing at company and customer levels.
4. Review start date, run hour, memo/terms/amount exclusions, receipt settings, and deposit-account mappings.
5. Test a known customer/invoice before broad rollout.
6. Use the forecast report/API to preview the next run.

## API-assisted forecast

The forecast date parameter is **`startDate`** (required), not `runDate`:

```text
GET /v2/autoprocessing-forecast?startDate=YYYY-MM-DD&limit=100&offset=0
GET /v2/autoprocessing-forecast?startDate=YYYY-MM-DD&invoiceId={invoiceId}
```

The response exposes:

- `willBeCharged`
- `reasons[]` (human-readable forecast outcome)
- `dueDateMet`
- `autoProcessingEnabled` (customer-level)
- `hasEnabledProfile`
- `startDateMet`
- `memoSkip`
- `amountSkip`
- installment status and `amountToCharge`

Also inspect:

```text
GET /v2/settings
GET /v2/payment-methods?customerId={customerId}
GET /v2/invoices/{invoiceId}
```

Use `GET /v2/settings` for company configuration and `GET /v2/payment-methods` for profile presence/status. Do not expose full payment details in summaries.

## Explain outcomes precisely

- `willBeCharged: true`: report the forecast amount and date; call it a forecast, not a guarantee of gateway approval.
- Customer autopay off: enable Auto Processing for the customer if the merchant intends to charge them.
- No enabled profile: request a payment method through Benji's secure portal/profile workflow; do not build a card form that sends card data to your server.
- Due/start date not met: explain when it becomes eligible.
- Memo, amount, terms, or installment rule: identify the matching setting and recommend a configuration change only after confirming intent.
- Decline: a decline happens at attempted payment time and is not the same as forecast ineligibility. Review transaction/gateway results and the payment profile; do not claim forecast `reasons` predicts issuer approval.

For declined charges, use transaction reporting (and `GET /v2/transactions` where appropriate) and follow gateway-specific remediation. Never retry a declined charge repeatedly without merchant review.

## Safety

- Auto Processing moves money. Before changing settings, enabling a customer, or initiating any charge, obtain explicit human approval for the exact scope.
- Start read-only with settings, forecast, profiles, and transactions.
- Never disclose or request raw card/bank credentials in chat.

## Official guides

- Auto Processing setup: https://support.benjipays.com/support/solutions/articles/150000180671-auto-processing-setup
- Customer Auto Processing behavior: https://support.benjipays.com/support/solutions/articles/150000185425-if-i-enable-auto-processing-for-a-customer-will-invoices-be-auto-paid-
- Forecast API: https://developer.benjipays.com/reference/get_v2-autoprocessing-forecast.md

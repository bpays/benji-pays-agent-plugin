---
name: embed-pay-now-and-portal
description: Configure or embed Benji Pays Pay Now links and the customer portal whenever a user wants a QBO, QBD, Xero, HaloPSA, Autotask, ConnectWise, Quoter, QuoteWerks, Datagate, PSA, CRM, ERP, website, invoice email, or PDF payment button; needs a stable customer payment link; sees invalid-link errors; or asks about portal domains, currencies, gateways, or guest checkout.
---

# Embed Pay Now and Customer Portal

Choose the link type by context. For invoice emails and templates, prefer stable links generated in Benji settings. API-created token links are short-lived and belong in authenticated, click-time application flows.

## Stable link for email/template workflows

Base pattern:

```text
https://www.benjipays.com/portal/{portalName}/pay/?InvoiceNumber={invoiceNumber}&transactionAmount={invoiceTotal}
```

Use the exact template shown in the merchant's Benji settings rather than reconstructing it when possible:

- **Settings → QuickBooks Custom Payment Links** for QBO templates
- **Settings → QuickBooks Desktop Custom Payment Links** for QBD
- **Settings → Custom Payment Links** for PSA/CRM/ERP/website workflows
- **Settings → Customer Portal Settings** for portal name and payment behavior

The customer portal is normally `https://{portalName}.benjipays.com`. A custom domain can be arranged through Benji Pays support.

## How to do common jobs

### QuickBooks Online invoice email

1. Connect the gateway in Benji Pays.
2. Open **Settings → QuickBooks Custom Payment Links**.
3. Choose the generic, currency-specific, or gateway-specific template.
4. Paste it into QBO **Account and settings → Sales → Messages → Invoice email**.
5. Keep the Benji-generated placeholders intact; QBO inserts the invoice number.
6. Test one invoice.

QBO cannot automatically insert the invoice amount in its email template, so do not enable an amount-required workflow for these links without validating compatibility.

### Xero invoice

Connect the Benji Pays payment service to a Xero branding theme under the gateway settings. Xero then presents the integrated payment option on invoice flows.

### PSA / CRM / ERP

If the product syncs invoices into QBO or Xero, use:

```text
PSA / CRM / ERP ↔ QuickBooks or Xero ↔ Benji Pays
```

Copy the stable link from Benji and replace its invoice number and total placeholders with that system's template tokens. Official support has first-class guides for HaloPSA, ConnectWise PSA, Datto Autotask, Quoter, QuoteWerks, and Datagate; the generic flow works for other systems that sync the invoice/customer into supported accounting.

- Generic PSA/ERP/CRM: https://support.benjipays.com/support/solutions/articles/150000181442-custom-payment-links-for-any-psa-or-erp-crm
- HaloPSA: https://support.benjipays.com/support/solutions/articles/150000211772-halopsa-invoice-pdf-integration
- Datto Autotask: https://support.benjipays.com/support/solutions/articles/150000211941-datto-autotask-invoice-pdf-integration
- ConnectWise PSA: https://support.benjipays.com/support/solutions/articles/150000223230-connectwise-psa-invoice-pdf-integration
- Quoter: https://support.benjipays.com/support/solutions/articles/150000211954-how-to-connect-quoter-to-benjipays
- QuoteWerks: https://support.benjipays.com/support/solutions/articles/150000212001-integrate-quotewerks-with-benji-pays
- Datagate: https://support.benjipays.com/support/solutions/articles/150000211944-datagate-integration

### QuoteWerks / QuoteValet deposits

QuoteWerks can expose a QuoteValet **Pay Now** button using a Benji custom payment link. This may be an **unapplied** payment because a quote is not yet an accounting invoice:

1. Copy the Benji link intended for a payment not tied to an invoice.
2. Map QuoteWerks/QuoteValet amount, document, and customer tokens.
3. After payment, locate it in Benji under **Transactions → Apply Payment**.
4. Apply it to the accounting invoice once that invoice exists.

Do not tell users that every QuoteValet payment automatically applies to an invoice. Whether it is invoice-tied or unapplied depends on the chosen link and whether the accounting invoice exists.

## Settings/API context

```text
GET /v2/settings
```

Inspect `data.customerPortal.name` and `data.customerPortal.url` for the configured portal identity. Controls include `payNowAmountRequired`, `disableGenericPaymentLink`, `customDomain`, and other portal behavior. Do not guess a merchant's portal name or gateway/currency template.

## API-created links: click-time only

```text
POST /v2/payment-links/applied/{invoiceId}
POST /v2/payment-links/unapplied
```

Both return `url` and `expiresAt`. Applied links use the accounting-system invoice ID, not the displayed invoice number. These tokenized URLs are short-lived (about an hour):

- Create them only at click time for an in-app/authenticated flow.
- **Never paste or automate them into email/SMS/customer message templates.** They may expire before the customer opens the message.
- For guest checkout, send `allowSavedPaymentMethods: false` (the default) unless the application independently authenticated the payer and established authorization to use that invoice customer's saved methods.
- Use an `Idempotency-Key` for safe POST retries.

## Invalid-link checklist

Check these before writing code:

1. Invoice is already paid, missing, duplicated by invoice number, or not synced yet.
2. Portal name is wrong or the organization is disabled.
3. No enabled gateway supports the invoice currency.
4. Multiple gateways support the same currency but a generic link was used; choose a gateway-specific link.
5. Multiple currencies require a currency-specific path/template.
6. `payNowAmountRequired` is on and `transactionAmount` is missing or does not equal the invoice total.
7. `disableGenericPaymentLink` or related portal controls disallow the chosen path.

Official troubleshooting: https://support.benjipays.com/support/solutions/articles/150000185071-invalid-link-error-on-pay-now-links

## Communication rule

Do not send a link to a customer. Draft the exact message and stable link, show recipients, and wait for explicit human approval. The Merchant API does not send email.

## Official guides

- QBO Pay Now: https://support.benjipays.com/support/solutions/articles/150000022845-quickbooks-online-integrated-pay-now-links
- Xero Pay Now: https://support.benjipays.com/support/solutions/articles/150000022627-xero-integrated-pay-now-links-setup
- Transaction amount setting: https://support.benjipays.com/support/solutions/articles/150000185669-require-transaction-amount-on-pay-now-invoice-links-to-match-the-invoice-s-total
- Portal configuration: https://support.benjipays.com/support/solutions/articles/150000210217-configure-customer-portal-settings
- Custom portal domains: https://support.benjipays.com/support/solutions/articles/150000197920-custom-domain-for-your-customer-portal

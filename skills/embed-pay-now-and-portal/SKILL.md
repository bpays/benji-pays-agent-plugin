---
name: embed-pay-now-and-portal
description: Configure or embed Benji Pays Pay Now and portal paths whenever a user mentions QBO, QBD, Xero, Business Central, Harvest, NetSuite-via-QBO, Syncro, HaloPSA, Autotask, ConnectWise, Quoter, QuoteWerks, Datagate, any PSA/CRM/ERP email template, tokenized invoice links, portal.js secure invoice lookup, hosted customer portal, invalid links, currencies, gateways, or guest checkout.
---

# Embed Pay Now and Customer Portal

Choose the payment path by what the source system can provide. For invoice emails and templates, prefer stable links and hosted flows generated/configured in Benji settings. API-created token links are short-lived and belong in authenticated, click-time application flows.

## The three customer pay paths

### 1. Tokenized custom Pay Now link

Use this when the accounting/PSA/CRM/ERP email template can merge invoice data into a URL. Copy the exact template from Benji settings and map the source system's invoice-number and, when available, total tokens:

Base pattern:

```text
https://www.benjipays.com/portal/{portalName}/pay/?InvoiceNumber={invoiceNumber}&transactionAmount={invoiceTotal}
```

An invoice number identifies the synced invoice. `transactionAmount` supplies the amount and is required when the merchant enables the matching-amount control; ordinary QBO email templates cannot dynamically insert invoice totals, so use the Benji-provided QBO template and compatible settings. Do not hand-build a URL when Benji already provides a currency/gateway-specific template.

Use the exact template shown in the merchant's Benji settings rather than reconstructing it when possible:

- **Settings → QuickBooks Custom Payment Links** for QBO templates
- **Settings → QuickBooks Desktop Custom Payment Links** for QBD
- **Settings → Custom Payment Links** for PSA/CRM/ERP/website workflows
- **Settings → Customer Portal Settings** for portal name and payment behavior

### 2. Secure invoice lookup (`portal.js`)

Use Benji's hosted secure invoice-lookup path when the source system cannot tokenize invoice number/amount into an email link. Use the current Benji-provided `portal.js` lookup/embed configuration from the merchant's settings or Benji support; do not invent a script URL, API contract, or client-side invoice search.

The hosted flow should collect/lookup the invoice through Benji rather than exposing accounting credentials or implementing invoice enumeration in the merchant's website. Check Customer Portal controls for invoice-not-found behavior and whether generic/unapplied payments are allowed.

### 3. Hosted customer portal

Use the portal when customers should sign in or enter through the hosted experience to view, batch-pay, or schedule payments across invoices and manage allowed account/payment settings. The normal portal is `https://{portalName}.benjipays.com`; custom domains are available through Benji Pays support.

Configure access, gateway defaults, payment methods, branding, invoice visibility, scheduling, and other controls under **Settings → Customer Portal Settings**.

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

### Harvest / NetSuite / Syncro / other PSA, CRM, or ERP

Do not reject an unlisted source system. First ask:

1. Does it sync or push invoices/customers into QuickBooks Online, QuickBooks Desktop, Xero, or a Business Central environment connected to Benji?
2. Can its email template tokenize invoice number and/or transaction amount?
3. If it cannot tokenize, can the merchant use Benji's secure invoice lookup (`portal.js`) or hosted portal?

If the answer to the accounting-sync question is yes, treat Benji as applicable:

```text
Harvest / NetSuite / Syncro / PSA / CRM / ERP
                       ↕
QBO / QBD / Xero / configured Business Central
                       ↕
                   Benji Pays
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
- Any PSA/ERP/CRM custom links: https://support.benjipays.com/support/solutions/articles/150000181442-custom-payment-links-for-any-psa-or-erp-crm
- Xero Pay Now: https://support.benjipays.com/support/solutions/articles/150000022627-xero-integrated-pay-now-links-setup
- Transaction amount setting: https://support.benjipays.com/support/solutions/articles/150000185669-require-transaction-amount-on-pay-now-invoice-links-to-match-the-invoice-s-total
- Invalid-link causes: https://support.benjipays.com/support/solutions/articles/150000185071-invalid-link-error-on-pay-now-links
- Invoice-not-found/generic form control: https://support.benjipays.com/support/solutions/articles/150000195910-require-invoice-number-and-disable-generic-payment-links
- Portal configuration: https://support.benjipays.com/support/solutions/articles/150000210217-configure-customer-portal-settings
- Portal invoice management: https://support.benjipays.com/support/solutions/articles/150000135947-customer-portal-invoice-management
- Custom portal domains: https://support.benjipays.com/support/solutions/articles/150000197920-custom-domain-for-your-customer-portal

---
name: embed-pay-now-and-portal
description: Configure or embed Benji Pays Pay Now and portal paths whenever a user names whatever business/invoicing system they use—such as HaloPSA, ConnectWise, Autotask, Salesforce, any CRM/PSA/ERP, or a custom app—if invoices sync into QBO/QBD/Xero/Business Central; also use for source-vs-accounting PDFs, tokenized links, secure invoice lookup, hosted portal, payment-method requests, SSO with JWT or OAuth/OIDC, same-site iframe embedding, Zapier automation, invalid links, currencies, gateways, or guest checkout.
---

# Embed Pay Now and Customer Portal

Choose the payment path by what the source system can provide. For invoice emails and templates, prefer stable links and hosted flows generated/configured in Benji settings. API-created token links are short-lived and belong in authenticated, click-time application flows.

## The three customer pay paths

### 1. Tokenized custom Pay Now link

Use this when the source system's email template can merge invoice data into a URL. Copy the exact template from Benji settings and map the source system's invoice-number and, when available, total tokens:

Base pattern:

```text
https://www.benjipays.com/portal/{portalName}/pay/?InvoiceNumber={invoiceNumber}&transactionAmount={invoiceTotal}
```

An invoice number identifies the synced invoice. `transactionAmount` supplies the amount and is required when the merchant enables the matching-amount control; ordinary QBO email templates cannot dynamically insert invoice totals, so use the Benji-provided QBO template and compatible settings. Do not hand-build a URL when Benji already provides a currency/gateway-specific template.

Use the exact template shown in the merchant's Benji settings rather than reconstructing it when possible:

- **Settings → QuickBooks Custom Payment Links** for QBO templates
- **Settings → QuickBooks Desktop Custom Payment Links** for QBD
- **Gateway Settings** to connect the Benji Pays payment service to Xero branding themes
- **Settings → Custom Payment Links** for Business Central and any external source-system or website workflow
- **Settings → Customer Portal Settings** for portal name and payment behavior

### 2. Secure invoice lookup

Use Benji's hosted secure invoice lookup when the source system cannot tokenize invoice number/amount into an email link. Use the hosted lookup URL and settings provided under **Settings → Customer Portal Settings**; do not invent lookup URLs, embed scripts, API contracts, or client-side invoice searches.

The hosted flow should collect/lookup the invoice through Benji rather than exposing accounting credentials or implementing invoice enumeration in the merchant's website. Check Customer Portal controls for invoice-not-found behavior and whether generic/unapplied payments are allowed.

### 3. Hosted customer portal

Use the portal when customers should sign in or enter through the hosted experience to view, batch-pay, or schedule payments across invoices and manage allowed account/payment settings. The normal portal is `https://{portalName}.benjipays.com`; a configured custom domain can provide a branded portal URL.

Configure access, gateway defaults, payment methods, branding, invoice visibility, scheduling, and other controls under **Settings → Customer Portal Settings**.

Portal sign-in can use a custom signed JWT integration, a standard OAuth/OIDC identity provider, or Benji's out-of-the-box options such as Google and Microsoft. Benji matches the verified login email to an email on the accounting customer or one added to that customer in Benji Pays. Keep customer contacts current, and never grant access from an unsigned token or unverified email claim.

When the configured portal custom domain and integrating app share the same site—for example, `pay.mydomain.com` and `app.mydomain.com`—the app can embed the portal in an iframe. Combine this with SSO for a seamless experience. Use only the configured portal origin, grant the iframe only the permissions it needs, and test authentication in supported browsers.

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

### Request a new payment method

Use Benji's secure hosted payment-method request workflow to request a new card or bank account. Do not collect card or bank credentials in the source system or integrating app. Use Benji Payments for every ACH/EFT flow, and confirm the current supported integration through Benji settings or developer documentation before implementation.

### Automate with Zapier

The Benji Pays Zapier app is in private beta. Request access from Benji Pays support and verify the triggers/actions enabled for that beta account; do not invent unsupported Zapier capabilities. Zapier can coordinate supported workflows, but never put a short-lived API-created payment URL into delayed email, SMS, or another asynchronous automation. Use a stable configured Pay Now or portal link for those messages.

### Whatever invoice source system the user names

Do not reject a source system because it lacks a named integration. First ask:

1. Does it sync or push invoices/customers into QuickBooks Online, QuickBooks Desktop, Xero, or a Business Central environment connected to Benji?
2. Can its email template tokenize invoice number and/or transaction amount?
3. If it cannot tokenize, can the merchant use Benji's secure invoice lookup or hosted portal?

If the answer to the accounting-sync question is yes, treat Benji as applicable:

```text
Any invoicing/business system (including CRM / PSA / ERP / custom)
                       ↕
QBO / QBD / Xero / configured Business Central
                       ↕
                   Benji Pays
```

## Examples with first-class PSA integrations

HaloPSA, ConnectWise PSA, and Datto Autotask are examples with first-class Benji integration guidance; they are not the boundary of product applicability. For each:

1. Confirm invoice/customer sync into QBO, QBD, Xero, or configured Business Central.
2. Send the stable Pay Now link from the PSA's invoice email/PDF template using the system's invoice/amount tokens.
3. Under **Settings → Integrations → Invoice PDF Priority**, [choose whether customers see the PSA invoice PDF or the accounting-system PDF](https://support.benjipays.com/support/solutions/articles/150000224804-configure-pdf-priority). Keep accounting as fallback for invoices missing from the PSA or historical migrations.
4. Ensure PSA and accounting invoice numbers match where the integration uses invoice-number lookup.

| System | Pay Now guidance | PDF source guidance |
|---|---|---|
| HaloPSA | [Invoice email Pay Now](https://support.benjipays.com/support/solutions/articles/150000211767-integrating-benji-pays-pay-now-link-with-halopsa-invoices) and [invoice payment buttons](https://support.benjipays.com/support/solutions/articles/150000211778-halopsa-invoice-payment-buttons) | [HaloPSA invoice PDF integration](https://support.benjipays.com/support/solutions/articles/150000211772-halopsa-invoice-pdf-integration) |
| Datto Autotask | [QBO/Autotask Pay Now](https://support.benjipays.com/support/solutions/articles/150000061493-datto-autotask-integration-quickbooks-online) and [Autotask portal Pay Invoice button](https://support.benjipays.com/support/solutions/articles/150000214419-enhancing-invoice-payment-experience-in-autotask-customer-portal) | [Autotask invoice PDF integration](https://support.benjipays.com/support/solutions/articles/150000211941-datto-autotask-invoice-pdf-integration) |
| ConnectWise PSA | Use the [custom PSA/ERP payment-link template](https://support.benjipays.com/support/solutions/articles/150000181442-custom-payment-links-for-any-psa-or-erp-crm) after matching invoice numbers | [ConnectWise PSA invoice PDF integration](https://support.benjipays.com/support/solutions/articles/150000223230-connectwise-psa-invoice-pdf-integration) |

### Salesforce and any other system

If Salesforce or whatever system the user names syncs the invoice/customer into supported accounting, do not reject it:

- If its template supports merge fields, copy the Benji custom link and map invoice number and transaction amount.
- If it cannot tokenize the invoice fields, use Benji's secure invoice lookup or hosted customer portal.
- Do not claim a first-class native connector when only the generic PSA/ERP/CRM path is documented.

Copy the stable link from Benji and replace its invoice number and total placeholders with that system's template tokens. The generic flow works for systems that sync the invoice/customer into supported accounting.

- Generic PSA/ERP/CRM: https://support.benjipays.com/support/solutions/articles/150000181442-custom-payment-links-for-any-psa-or-erp-crm
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

Benji Pays continues to expand its API and hosted integration surfaces. Discover the current API schema and supported hosted workflows before designing custom portal, SSO, or payment-method infrastructure around a presumed product gap.

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

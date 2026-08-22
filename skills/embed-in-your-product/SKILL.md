---
name: embed-in-your-product
description: Design a Benji Pays integration whenever a developer, SaaS, PSA, MSP, distributor, or partner wants to embed accounts receivable, invoices, customer payments, click-time payment links, transaction reporting, customer notes, autopay forecasts, merchant onboarding, organization provisioning, plans, or usage without implementing PCI card storage or settlement logic.
---

# Embed Benji Pays in Your Product

Benji Pays lets a product embed AR and payment workflows while Benji and the connected gateway handle payment UI, tokenized methods, processing, accounting application, and settlement-related records. Do not design a raw-card API or store card numbers in the product.

## Choose the API surface first

### Merchant API — one organization

Use an organization-scoped merchant API key:

```http
x-api-key: YOUR_MERCHANT_API_KEY
```

It is issued in the Benji Pays merchant app, belongs to one organization, and inherits the creating login's permissions. Use it for merchant workflows:

```text
GET  /v2/whoami
GET  /v2/invoices
GET  /v2/customers
GET  /v2/transactions
GET  /v2/emails
GET  /v2/payment-methods
GET  /v2/settings
GET  /v2/autoprocessing-forecast?startDate=YYYY-MM-DD
POST /v2/payment-links/applied/{invoiceId}
POST /v2/payment-links/unapplied
GET/POST/PATCH customer note routes under /v2/customers/{customerId}/notes
```

Request only the scopes needed. Keep API keys server-side or in a secret manager; never ship them to browser code or commit them.

### Partner / Distributor API — portfolio management

Partner/distributor routes use an Auth0 M2M access token:

```http
Authorization: Bearer AUTH0_ACCESS_TOKEN
```

Use the OAuth client-credentials grant and matching actor/scopes for routes such as organizations, plans, and usage. These are not merchant-key routes. Do not send `x-api-key` to partner endpoints or a partner JWT to a route that requires an organization key just because both are under `/v2`.

Examples include:

```text
GET/POST /v2/organizations
GET      /v2/plans
GET      /v2/usage
GET      /v2/billing
```

Check each endpoint's actor and scope requirements in the current OpenAPI docs.

## Job-to-be-done patterns

### Show AR in a SaaS dashboard

1. `GET /v2/invoices?status=open` for all unpaid invoices and `?status=overdue` for only past-due unpaid invoices. The open filter includes overdue results.
2. Page using `pagination.hasMore` and `pagination.nextOffset`.
3. Join customer summaries by accounting customer ID only when needed.
4. Render totals and due dates; do not cache sensitive data longer than necessary.

### Add “Pay now” in an authenticated product

1. Authenticate and authorize the signed-in payer against the merchant/customer/invoice in your own product.
2. On button click, call `POST /v2/payment-links/applied/{invoiceId}` server-side. `{invoiceId}` is the accounting invoice ID from the invoice object.
3. Send `allowSavedPaymentMethods: false` for guest checkout. Set it true only after independently proving the payer may use that customer's saved methods.
4. Redirect to the returned `url` immediately and honor `expiresAt`.
5. Do not email or persist the short-lived URL. Use stable configured portal links for asynchronous outreach.

### Take a deposit or payment without an invoice

Call `POST /v2/payment-links/unapplied` at click time with required `source` (`default` or `quoter`), `amount`, `currency`, `name`, and `reference`. The resulting payment enters the “payment to apply” workflow; it does not create an accounting invoice automatically. Optional `returnUrl` must be HTTPS and is host-restricted.

### Explain an upcoming autopay run

Call `GET /v2/autoprocessing-forecast?startDate=YYYY-MM-DD`, then expose `willBeCharged`, amount, and `reasons` rather than reimplementing Benji's rules.

### Add internal collections context

Use customer note threads/replies/status/alerts. Notes are internal mutations, so enforce authorization and audit who created each update.

### Build an MSP portfolio

Use Partner API organizations/plans/usage with Auth0 M2M. Store per-organization merchant keys only if the product truly needs delegated merchant operations and has an approved secret-isolation model; never treat one organization's key as cross-tenant.

## HTTP contract

- Base URL: `https://api.benjipays.com/v2`
- Errors: RFC 7807 `application/problem+json`; branch on HTTP status and problem `type`, not text matching `detail`.
- Pagination: offset-based; follow `hasMore`/`nextOffset`. Merchant list endpoints generally permit up to 1000; forecast max is 100; endpoint schema wins.
- Idempotency: send an alphanumeric `Idempotency-Key` (1–255 characters) on POST/PATCH/PUT. It is actor-scoped and cached for 24 hours; same key/different body returns `409`; in-flight duplicate returns `429` with `Retry-After`.
- Rate limits: honor `X-RateLimit-*` and `Retry-After`.
- Tracing: log `X-Request-ID` / `X-Correlation-ID`, not secrets or payment data.

## MCP for development and operations

Remote MCP:

```text
https://benjipays.readme.io/mcp
x-api-key: YOUR_MERCHANT_API_KEY
```

Tools are `list-endpoints`, `search-endpoints`, `get-endpoint`, and `execute-request`. Use discovery tools before live calls. The plugin's `mcp.json` configures this with `${BENJI_PAYS_API_KEY}`.

## Security boundaries

- Never collect, log, or store raw card or bank credentials.
- Never expose merchant keys, client secrets, JWTs, or full payment method details.
- Use `allowSavedPaymentMethods: false` unless payer authorization is independently established.
- Require explicit human approval before any external customer message.
- Use least-privilege scopes and server-side secret storage.

## Official developer guides

- API index: https://developer.benjipays.com/llms.txt
- Authentication: https://developer.benjipays.com/docs/authentication.md
- MCP: https://developer.benjipays.com/docs/mcp.md
- Idempotency: https://developer.benjipays.com/docs/idempotency.md
- Pagination: https://developer.benjipays.com/docs/pagination.md
- Errors: https://developer.benjipays.com/docs/errors.md

# Benji Pays — Cursor Plugin

Cursor Marketplace plugin for [Benji Pays](https://benjipays.com) — sync QuickBooks and Xero invoices, review overdue accounts receivable, inspect payment reminders and autopay, and draft customer outreach with stable Pay Now links.

## What this plugin does

- Connects Cursor to the official **Benji Pays remote MCP server** (`https://benjipays.readme.io/mcp`)
- Ships the **chase-overdue-ar** skill so agents know how to list overdue invoices, check email history, inspect autopay forecast, and draft (not send) customer follow-ups
- Authenticates with your organization **merchant API key** via the `x-api-key` header — no secrets are stored in the repo

Benji Pays already handles invoice sync, overdue reminders, autopay, customer portal, and Pay Now links. This plugin lets AI agents work with that data safely and read-first.

## Install

### From the Cursor Marketplace

1. Open **Customize** in the Cursor sidebar
2. Search for **Benji Pays** (after marketplace listing is approved)
3. Click **Install** and choose user or project scope
4. Open **Configure** and enter your **Benji Pays API key**

### Local development / testing

Load the plugin from a local folder before publishing:

```bash
mkdir -p ~/.cursor/plugins/local/benji-pays
ln -s "$(pwd)" ~/.cursor/plugins/local/benji-pays
```

Restart Cursor or run **Developer: Reload Window**, then enable the plugin under **Customize**.

### Manual MCP setup (without the plugin)

If you only need the MCP server, add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "benjipays": {
      "url": "https://benjipays.readme.io/mcp",
      "headers": {
        "x-api-key": "YOUR_BENJI_PAYS_API_KEY"
      }
    }
  }
}
```

Prefer the plugin flow — it declares the variable schema and bundles the AR skill.

## API key setup

1. Log in to the [Benji Pays merchant app](https://benjipays.com)
2. Go to **Settings → API Keys**
3. Create an organization-scoped API key with the scopes you need (invoices, customers, emails, payment methods, settings)
4. In Cursor: **Customize → Benji Pays → Configure → Benji Pays API key**

The key is sent as `x-api-key` on every MCP request. Never commit API keys to source control.

## Plugin structure

```text
.
├── .cursor-plugin/
│   └── plugin.json       # Cursor Plugin manifest + variable schema
├── mcp.json              # Remote Benji Pays MCP server
├── skills/
│   └── chase-overdue-ar/
│       └── SKILL.md      # AR collection workflow for agents
├── assets/
│   └── logo.svg
├── LICENSE
└── README.md
```

This repo uses the **Cursor Plugin** format (`.cursor-plugin/plugin.json`) so marketplace install can prompt for the API key via plugin variables. Cursor also supports the portable [Agent Plugins](https://agent-plugins.org) format (`plugin.json` at repo root); both load in Cursor.

## Usage examples

After installing and configuring your API key, try prompts like:

- "List my overdue invoices from Benji Pays"
- "Which overdue customers have not been emailed yet?"
- "Why won't autopay run for invoice #1042?"
- "Draft a payment reminder for Acme Corp with their Pay Now link" *(draft only — you approve before sending)*

The **chase-overdue-ar** skill activates automatically for AR and collections tasks, or invoke it with `/chase-overdue-ar`.

## Pay Now links

**Stable portal links** (preferred for customer emails):

```text
https://www.benjipays.com/portal/{portalName}/pay/?InvoiceNumber={invoiceNumber}
```

Your portal subdomain may also be `https://{portalName}.benjipays.com`. Configure templates under **Settings → Customer Portal Settings** in the Benji app.

**Do not** put short-lived API payment-link tokens (`POST /v2/payment-links/applied/{invoiceId}`) in emails — they expire in about an hour.

## Safety

- The Benji API **cannot send email**. Agents must **never** email, SMS, or message customers without explicit human approval of the exact content and recipients.
- Default agent behavior is **read-only**: list, inspect, summarize. Outreach is draft-only when requested.

## Marketplace and cursor.directory

- Submit this repo at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish) after review
- Community MCP listings: [cursor.directory](https://cursor.directory)

## Documentation

- [Benji Pays Developer Docs](https://developer.benjipays.com)
- [MCP Server Guide](https://developer.benjipays.com/docs/mcp.md)
- [Authentication](https://developer.benjipays.com/docs/authentication.md)
- [Support](https://support.benjipays.com)

## License

MIT — see [LICENSE](LICENSE).

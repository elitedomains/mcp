# ELITEDOMAINS MCP Server

Register and transfer domains, back-order expiring `.de` domains, run sale pages,
answer buyers and read the invoices behind it all — from any AI agent that speaks the
[Model Context Protocol](https://modelcontextprotocol.io).

This is the official MCP server of **[ELITEDOMAINS](https://elitedomains.de)**, and it
is hosted by us. There is nothing to install, clone or run: point an MCP client at
`https://mcp.elitedomains.de`, authenticate with a personal access token from your
ELITEDOMAINS account, and the 39 tools below appear.

|  |  |
| --- | --- |
| **Endpoint** | `https://mcp.elitedomains.de` |
| **Transport** | Streamable HTTP |
| **Authentication** | Bearer token — the same personal access token as the REST API |
| **Tools** | 39 |
| **Server version** | 1.0.0 |
| **Price** | free; you pay for the domains you buy, not for the interface |
| **Operator** | [ELITEDOMAINS](https://elitedomains.de), Germany |

## Who we are

ELITEDOMAINS is an independent, owner-operated German domain provider. Two domainers
founded it in 2017 and opened it to everyone in 2018, because the control panels they
had to work with every day were built for people who own one domain, not a few thousand
— and the motto has not changed since: *von Domainern für Domainer*, by domainers for
domainers.

We write every line of the platform ourselves and run our own registry connections,
including a direct one to [DENIC](https://www.denic.de), the registry behind `.de`.
That is where the deep `.de` features come from:
[catching expiring `.de` domains](https://elitedomains.de/de-catcher), ordering an
[AuthInfo2 code](https://elitedomains.de/auth-info-2-code), dispute alerts and a DENIC
monitor. On top of that sits the part of the business our customers earn money with:
[selling domains](https://elitedomains.de/domains-verkaufen) through sale pages with an
escrow-secured instant purchase, a marketplace, and Sedo synchronisation.

The MCP server exposes that platform, tool for tool, to an agent working on your
behalf. It is the same account, the same portfolio and the same prices you see when you
log in — [see for yourself](https://elitedomains.de/preise).

## What you can do with it

- **Domains** — search and filter the portfolio, check availability and what an order
  would cost, register and transfer, change nameservers, redirector and owner, order
  AuthInfo / AuthInfo2, delete or hand a domain into transit, set tags, list on Sedo.
- **Owner handles** — list, create and update the contact handles behind the domains.
- **Sale pages** — create, price, update and remove the pages a domain is offered on.
- **Buyer leads** — read incoming enquiries, send a counter-offer, accept a suggested
  price.
- **Notes and journal** — the private notes on a domain, and the log of what changed.
- **Transactions and invoices** — past sales, invoices with their line items, and the
  service records behind a position.
- **Catcher** — order, review and cancel backorders for expiring `.de` domains.

All TLDs we offer are supported; the DENIC-specific features (catcher, AuthInfo2,
transit) are `.de` only. The `domain-prices` tool answers what a TLD costs.

## Getting started

**1. An ELITEDOMAINS account.** [Sign up](https://app.elitedomains.de/register) — free,
and you only pay for what you register.

**2. API access.** It is off until you ask for it: write to
[support](https://elitedomains.de/kontakt) or use the chat in the app, and we will
enable it. The [FAQ entry](https://elitedomains.de/faq/wie-funktioniert-api-schnittstelle)
explains what happens then.

**3. A personal access token.** Create one under
[Settings → API](https://app.elitedomains.de/settings#api). You choose its scopes when
you create it and cannot change them afterwards, so grant only what the agent should be
able to do — see [Scopes](#scopes) below. The token is shown once.

**4. Point your client at the server.**

### Claude Code

```bash
claude mcp add --transport http elitedomains https://mcp.elitedomains.de \
  --header "Authorization: Bearer YOUR_TOKEN"
```

### Claude Desktop, Cursor, VS Code and other HTTP-capable clients

```json
{
  "mcpServers": {
    "elitedomains": {
      "type": "http",
      "url": "https://mcp.elitedomains.de",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

### Clients that only speak stdio

Bridge the HTTP server into a local process with
[`mcp-remote`](https://www.npmjs.com/package/mcp-remote):

```json
{
  "mcpServers": {
    "elitedomains": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote", "https://mcp.elitedomains.de",
        "--header", "Authorization: Bearer YOUR_TOKEN"
      ]
    }
  }
}
```

Your token is a password to a portfolio that costs real money to change. Keep it out of
repositories and shared configuration files, and revoke it in
[Settings → API](https://app.elitedomains.de/settings#api) the moment you suspect it
leaked.

## Scopes

A token carries scopes of the form `<resource>:<access>`, for example `domains:read` or
`offers:write`. Every tool requires exactly one, listed in the tables below, and the
server only offers an agent the tools its token actually covers: authenticate with a
read-only token and no tool that changes anything is even visible.

Read and write are independent — `domains:write` does **not** imply `domains:read`.
Two broader grants exist as well: `read` covers every read scope and `write` every
write scope, including resources we add later.

Calling a tool the token has no scope for fails with a message naming the missing
scope, rather than doing half the work.

## Tools

Every tool mirrors one REST endpoint of the
[ELITEDOMAINS API](https://app.elitedomains.de/docs), which documents each parameter,
each response field and each error. Arguments, validation and results are identical —
whichever interface you use, you are talking to the same code.

Tools marked ⚡ execute an operation at a registry and count against the stricter of the
two rate limits (see [Rate limits](#rate-limits)).

### Domains

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-domains` | List the domains in the authenticated account, with filtering, ordering and pagination. | `GET /domains` | `domains:read` |
| `list-domain-filters` | List the tags and TLDs available to filter the domains listing by, with counts. | `GET /domains/filters` | `domains:read` |
| `check-domain` ⚡ | Check the registry status of a domain (free, connect, ...). Also returns the order preview fields of check-domain-order (action, periods, price, billing). | `GET /domains/check` | `domains:read` |
| `check-domain-order` ⚡ | Preview an order for a domain before placing it: registry status, resulting action (register/transfer), available periods with costs and how the order would be billed (instant vs. monthly invoice). | `GET /domains/order/check` | `domains:read` |
| `domain-prices` | Get domain registration and renewal prices, optionally filtered by a single TLD. | `GET /domains/prices` | `domains:read` |
| `register-domain` ⚡ | Register a new domain or transfer an existing one into the account. | `POST /domains` | `domains:write` |
| `update-domain` ⚡ | Update a domain's redirector settings and/or owner handle. | `PATCH /domains` | `domains:write` |
| `delete-domain` ⚡ | Delete a domain, or put it into the transit state instead of deleting. | `DELETE /domains` | `domains:write` |
| `set-domain-tags` | Assign tags to a domain in the account. | `POST /domains/tags` | `domains:write` |
| `get-domain-auth-info` ⚡ | Get the AuthInfo (transfer) code for a domain in the account. | `GET /domains/authinfo` | `domains:write` |
| `check-auth-info2` ⚡ | Check whether AuthInfo2 can be ordered for a domain (needed for some inbound transfers). | `GET /domains/authinfo2/check` | `domains:read` |
| `order-auth-info2` ⚡ | Order AuthInfo2 for a domain (required for certain inbound transfers). | `POST /domains/authinfo2` | `domains:write` |
| `sedo-add-domain` | List a domain for sale on Sedo at the given price. | `POST /domains/sedo` | `domains:write` |

### Handles

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-handles` | List the contact handles in the authenticated account (paginated). | `GET /handles` | `handles:read` |
| `create-handle` ⚡ | Create a new contact handle. Handles provide the domain contact information used at the registry. | `POST /handles` | `handles:write` |
| `update-handle` ⚡ | Update an existing contact handle. Only include the fields you want to change. | `PATCH /handles/{id}` | `handles:write` |

### Offers

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-offers` | List the sale pages (domain offers) in the account, with filtering, ordering and pagination. | `GET /offers` | `offers:read` |
| `list-offer-filters` | List the tags and TLDs available to filter the sale pages (offers) listing by, with counts. | `GET /offers/filters` | `offers:read` |
| `create-offer` | Create a sale page (domain offer) for a domain in the account. | `POST /offers` | `offers:write` |
| `update-offer` | Update the configuration of a sale page (domain offer) in the account, e.g. template, price, status, or content. | `PATCH /offers` | `offers:write` |
| `update-offer-price` | Update the asking price of a sale page (domain offer) in the account. | `PATCH /offers/price` | `offers:write` |
| `delete-offer` | Delete a sale page (domain offer) from the account. The domain itself is not affected. | `DELETE /offers` | `offers:write` |

### Leads

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-leads` | List the leads (interested buyers) across the account's sale pages, with filtering by domain, ordering and pagination. | `GET /leads` | `leads:read` |
| `send-counter-offer` | Send a counter-offer (new asking price) for a lead, or the seller's first price if the lead had none yet. | `POST /leads/{id}/counter-offer` | `leads:write` |
| `accept-price-suggestion` | Accept a lead's pending buyer price suggestion. | `POST /leads/{id}/accept` | `leads:write` |

### Notes

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-notes` | List the account's private domain notes, with filtering by domain, ordering and pagination. | `GET /notes` | `notes:read` |
| `update-note` | Write the account's private note for a domain, replacing any existing note on it. | `PATCH /notes` | `notes:write` |
| `delete-note` | Remove the account's private note from a domain. The domain itself is not affected. | `DELETE /notes` | `notes:write` |

### Journal

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-journal` | List the journal entries recording changes made to the account's domains, with filtering by domain, type and source, and pagination. | `GET /journal` | `journal:read` |

### Transactions

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-transactions` | List your domain sale transactions, with filtering by domain, sorting, and pagination. | `GET /transactions` | `transactions:read` |

### Invoices

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-invoices` | List the account's invoices with number, date, status, payment method and amounts. | `GET /invoices` | `invoices:read` |
| `get-invoice` | Get a single invoice including its line items (positions). | `GET /invoices/{id}` | `invoices:read` |
| `list-invoice-services` | List an invoice's service records (Leistungsabrechnung): one record per billed domain or service. | `GET /invoices/{id}/services` | `invoices:read` |

### Catcher

| Tool | What it does | REST | Scope |
| --- | --- | --- | --- |
| `list-catcher-orders` | List the backorder (catcher) orders in the authenticated account, with filtering, ordering and pagination. | `GET /catcher` | `catcher:read` |
| `list-quarantine` | List .de quarantine domains available for backorder, with deadlines (earliest drop days), rating, DNS usage of other TLDs, bidder counts and whether you have added them. | `GET /catcher/quarantine` | `catcher:read` |
| `list-catcher-filters` | List the tags and TLDs available to filter the backorder (catcher) orders listing by, with counts. | `GET /catcher/filters` | `catcher:read` |
| `catcher-info` | Get backorder (catcher) information for a specific domain name. | `GET /catcher/info` | `catcher:read` |
| `create-catcher-order` ⚡ | Place a backorder (catcher) for a .de domain so it is caught when it drops. Only .de domains are supported. | `POST /catcher` | `catcher:write` |
| `delete-catcher-order` | Remove a backorder (catcher) order for a domain. | `DELETE /catcher` | `catcher:write` |

## Paid actions

Registering a domain, transferring one in and catching a `.de` domain cost money, and
an agent can trigger all three. They are billed exactly as they are in the web app:
through the monthly collective invoice if your account has monthly billing for that
TLD, and otherwise charged **instantly** to your stored payment method. An order is
rejected when there is no usable payment method.

`check-domain-order` answers what an order would do, which periods are available, what
it would cost and how it would be billed — without ordering anything. It is the tool an
agent should reach for before `register-domain`.

## Rate limits

Two limits apply per account, and REST and MCP share the counters:

- Tools that reach a registry (marked ⚡ above): **100 requests/minute, 1,000/day**.
  Between 02:00 and 04:00 (Europe/Berlin), DENIC's droptime, both ceilings drop
  tenfold, to **10/minute and 100/day** — and the daily one applies to the same
  counter you have been filling since midnight, it is not a fresh allowance for the
  window. An account that has already made 100 registry calls before 02:00 has nothing
  left for the droptime; plan the day around the window rather than the other way
  round.
- Everything else: **600 requests/minute, 20,000/day**, not reduced during droptime.

Those are the defaults — talk to us if your workload needs more.

## Sandbox mode

Every tool takes a `sandbox` argument. With `sandbox: 1` the call is authenticated and
validated exactly as usual, but nothing is written, nothing is ordered and nothing is
charged — the safe way to let a new agent loose on a real portfolio for the first time.

**Judge a dry run by what you sent, not by what came back.** Most tools mark it in
their answer ("Note would be saved (sandbox mode)"), but not all of them do:
`register-domain` replies `Domain created` either way. Nothing was created — the
message is simply the same one a real registration returns, so neither you nor an agent
can tell the two apart from the response alone.

The tools' input schemas do not list the argument, so a model will not reach for it on
its own: put it in your client's instructions ("always call the ELITEDOMAINS tools with
sandbox: 1") for as long as you want a dry run.

## Privacy

The server acts strictly on the account the token belongs to and returns only that
account's data. What an MCP client sends to a model, and what its operator does with
it, is between you and that client — nothing about your portfolio leaves ELITEDOMAINS
until an agent you configured asks for it. See our
[privacy policy](https://elitedomains.de/datenschutz).

## Support

- **Questions about the server or the API** — [contact us](https://elitedomains.de/kontakt),
  or use the chat in the app; we answer our support ourselves.
- **Bugs and feature requests** — open an issue in this repository.
- **API reference** — [app.elitedomains.de/docs](https://app.elitedomains.de/docs),
  MCP section: [MCP Server (AI Agents)](https://app.elitedomains.de/docs#mcp-server-ai-agents).

## ELITEDOMAINS

[Home](https://elitedomains.de) ·
[Prices](https://elitedomains.de/preise) ·
[Sell domains](https://elitedomains.de/domains-verkaufen) ·
[.de Catcher](https://elitedomains.de/de-catcher) ·
[AuthInfo2](https://elitedomains.de/auth-info-2-code) ·
[FAQ](https://elitedomains.de/faq) ·
[Changelog](https://elitedomains.de/changelog) ·
[About us](https://elitedomains.de/about) ·
[Contact](https://elitedomains.de/kontakt) ·
[Imprint](https://elitedomains.de/impressum) ·
[Privacy](https://elitedomains.de/datenschutz) ·
[Terms](https://elitedomains.de/agb)

---

<sub>Generated from the live tool catalogue of the ELITEDOMAINS MCP server
(version 1.0.0) with `php artisan mcp:docs`. The server itself is
closed-source and hosted by us; this repository only carries its documentation.</sub>

# Furniture Guild

A 2nth.ai front-end demo, showing what modern, purpose-built UIs look like on top of a private ERPNext instance.

## What this is

Two pages — the same data, two very different jobs:

| Page | URL | Who it's for |
|------|-----|-------------|
| [`index.html`](./index.html) | [furnitureguild.pages.dev](https://furnitureguild.pages.dev) | Customers — browse the catalogue, request a quote, accept and pay |
| [`workshop.html`](./workshop.html) | [furnitureguild.pages.dev/workshop](https://furnitureguild.pages.dev/workshop.html) | The workshop team — active work orders, stock, schedule, alerts |

Both are styled distinctly — warm wood tones for the customer-facing storefront, a dense dark operational layout for the workshop — but they share the same design primitives (Outfit + JetBrains Mono, dark/light toggle, wood/sage/sky palette).

## The 2nth pattern

ERPNext is operationally brilliant and visually brutal. Most South African SMEs that need a proper SoR (sales, stock, accounting, manufacturing) end up either living with its default UI, paying for a hosted SaaS that abstracts it, or rebuilding parts in spreadsheets.

The 2nth pattern is the third path: **keep ERPNext as the system of record, wrap it in views that are built for the job.** Consumer-facing, ops-facing, exec-facing — all different front-ends, all talking to the same private ERPNext API through a Cloudflare Worker at the edge.

## Architecture (this demo and where it's headed)

### Current state — static demo

Both pages are currently static HTML with baked data that matches a real seeded ERPNext instance (16 products, 12 SA retail customers, 5 suppliers, active work orders, raw + finished-goods stock, an AR aging set). The data is **consistent across both pages** — when the storefront says Oak Dining Table is R12,000, the workshop shows the same unit value in stock reporting. Same for orders, schedules, and alerts.

### Phase 2 — wired to live ERPNext

```
Customer browser  →  Cloudflare Pages (this repo, static)
                  →  Cloudflare Worker (signs JWT with GCP SA key)
                  →  Private ERPNext on Cloud Run in africa-south1
                  →  Cloud SQL · Redis · Cloud Storage
```

The Worker mints a Google OIDC ID token from a service-account key stored as a Worker secret, calls the private Cloud Run service (which fronts ERPNext), and proxies the response back. No public IP on ERPNext, no CORS theatre, no SaaS middleman.

See:
- **Infrastructure:** [2nth × Google Cloud — Johannesburg, operational](https://know-2nth.pages.dev/explainers/tech/google/demo.html)
- **Compute layer:** [2nth compute tier — Cloud Run, GCE, GKE in africa-south1](https://know-2nth.pages.dev/explainers/tech/google/compute.html)
- **Canonical skill nodes:** [github.com/2nth-ai/skills — `tech/google/*`](https://github.com/2nth-ai/skills/tree/main/tech/google)

## Running locally

```bash
# Any static server will work
npx serve .
open http://localhost:3000
```

Or with Wrangler (matches the deploy target):

```bash
npx wrangler pages dev .
```

## Deploying

Uses Cloudflare Pages via the Wrangler CLI. One-liner once authenticated:

```bash
npm run deploy
```

Which runs:

```bash
wrangler pages deploy . --project-name=furnitureguild --branch=main --commit-dirty=true
```

Preview branches get their own URL automatically — e.g. `feat-checkout.furnitureguild.pages.dev`.

## Where this is going

Short list, roughly in build order:

1. **Worker + Tunnel** — Cloudflare Worker proxying real ERPNext API calls; Cloudflare Tunnel exposing the private instance
2. **Real quote submission** — storefront form → Worker → ERPNext `Quotation` API → confirmation email via Mailchannels
3. **Live stock** — workshop dashboard reads real stock from ERPNext, auto-refreshes every N seconds
4. **Customer login** — Cloudflare Access for customers to track their orders after quote acceptance
5. **Payment integration** — PayFast / Ozow / Stripe webhook → ERPNext Payment Entry
6. **Mobile workshop tablet** — cut-down dashboard optimised for a workshop wall-mounted iPad
7. **AI assist** — Gemini / Claude via Vertex AI for quote description drafting, customer email responses, stock-level forecasting

## Contributing

This is a 2nth.ai asset used to demonstrate the pattern for real client engagements. If you're a 2nth team member, open a PR against `main`. External contributions welcome for obvious bugs and accessibility improvements.

## License

MIT — see [LICENSE](./LICENSE).

## Stack

- Pure HTML + CSS + vanilla JS (no build step)
- Outfit (body) + JetBrains Mono (code/labels) — Google Fonts
- Cloudflare Pages for hosting
- Cloudflare Workers planned for the live-API layer
- ERPNext v16 on Google Cloud Run in `africa-south1` as the SoR

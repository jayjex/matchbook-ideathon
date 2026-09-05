# Matchbook — The settlement-truth layer for Solana payments

**IDEATHON submission · Superteam Ukraine · Colosseum Hackathon idea · September 2026**

> Every transfer knows its invoice.

**Deliverables:** [`deck.pdf`](./deck.pdf) (10-slide pitch deck, same content as this README) · this document

---

## problemStatement

Solana moves ~$650B/month in adjusted stablecoin volume (Artemis/Everstake, Feb 2026) with ~6M monthly USDC senders — yet payment bookkeeping on-chain is still manual. The ecosystem itself names the problem:

> "Businesses using crypto for accounts payable and receivable face a reconciliation problem: verifying which invoice corresponds to which on-chain transfer."
> — Solana Compass, Request Network profile (solanacompass.com/projects/request-network)

Concretely, anyone paying contractors, vendors, or APIs in USDC today:

- pays via Streamflow batch payouts / Sphere billing / a raw transfer, then **manually** matches each on-chain transfer to an invoice in a spreadsheet or QuickBooks;
- has **no protocol-level link** between transfer and invoice — memos are optional, wallets strip them, and nothing enforces the binding;
- has **no partial-payment, refund, or dispute primitives** — a stablecoin payment is final, push-based, and its "paper trail" is a screenshot;
- and in the emerging agent economy (x402: 167M+ settled transactions by May 2026; Pay.sh; Circle Agent Stack), **machines pay with zero receipts and zero recourse** — the x402 spec defines no refunds, disputes, or receipt binding, and the x402 Foundation's public ROADMAP.md is an empty placeholder (docs.x402.org; github.com/x402-foundation/x402).

Who experiences it: DAO ops teams (Streamflow alone serves 24,000+ projects), crypto-native companies and agencies paying global contractors in USDC (Gusto added same-day USDC contractor payouts in March 2026), API businesses selling to AI agents (Cloudflare x402 workers, MCP tool marketplaces), and the agents themselves.

## technicalApproach

Matchbook is three boring, buildable components on proven Solana primitives:

**1. Proof-of-Invoice binding — Anchor program (Rust, ~600 LOC).**
A Token-2022 **transfer-hook program** plus a registry PDA. To accept a payment on a Matchbook-configured mint, the transfer must carry a canonical invoice hash that already exists in the registry — the invoice↔transfer binding becomes a **protocol property enforced at settlement**, not a convention typed in afterwards. This is native to Solana's Token-2022 extension and impossible to retrofit on EVM. Fallback mode for non-hook mints: Solana Pay memo-reference parsing.

**2. Auto-match engine — indexer + matcher.**
A Geyser/websocket indexer streams bound transfers into Postgres. A deterministic matcher reconciles them against the user's ledger (QuickBooks/Xero CSV export): exact invoice-hash match first, amount+payee heuristic second, confidence score surfaced. Exposed as a web app, REST API, and webhooks. On Solana's ~400ms settlement, reconciliation happens *with* the payment, not weeks later.

**3. Dispute & refund rails — escrow PDA state machine.**
A program-derived escrow account per invoice tracks: `partial → complete → disputed → refunded`. Partial payments accrue against the invoice total; refunds are signed claims back to source; disputes freeze the state and attach an evidence hash. Deliberately minimal — no oracle, no court. The same rails give x402-style agent purchases their missing recourse path.

**Hackathon build plan (5 days, devnet, open source):**
Day 1–2 transfer-hook program + local validator tests → Day 3 indexer + matcher demo → Day 4 QuickBooks CSV round-trip → Day 5 x402 receipt demo (agent buys an API call; receipt reconciles itself).

No new consensus, no oracle dependency, no proprietary protocol — all components reuse shipping Solana building blocks (Token-2022 hooks since 2024, Geyser plugins, Anchor).

## targetAudience

**First user is the author.** I operate AI agents that pay for API services weekly; Matchbook becomes their expense ledger at MVP. The idea exists because I hit the problem personally — agents with a daily spend mandate currently produce no matched, auditable expense record.

**Beachhead segments (in order):**
1. **DAO ops & crypto-native companies** — Streamflow's 24,000+ projects pay contributor batches today with CSV-export-and-eyeballs reconciliation. Matchbook plugs in as a webhook from day one.
2. **Agencies & offshore teams** — stablecoin contractor pay is mainstream (Gusto USDC payouts Mar 2026; Sphere recurring billing GA Feb 2026); their finance teams inherit the reconciliation problem at scale.
3. **API businesses serving agents** — MCP/x402 API sellers need receipt + refund rails before serious buyers trust agent-driven spend.
4. **The agent economy** — an agent with a $50/day mandate can't produce a matched expense log today. Same primitives, new customer.

## businessModel

**Priced like the spreadsheet it replaces:**

- **Free tier:** 50 matched transfers/month, devnet, public API.
- **Team — $49/month:** unlimited matching, QuickBooks/Xero sync, webhooks, dispute console.
- **Enterprise:** custom mints, private indexer, SOC 2 track. (Reference: Request Finance charges $300–1,500/mo for the broader suite — room to undercut with focus.)
- **Dispute-rail fee:** 0.1% of value settled through escrow PDAs (refunds, partial payments, agent-receipt flows).

**Why it compounds:** the matched-ledger data becomes the accounting system of record for stablecoin-native businesses and the only structured expense data source in the agent economy — a data network effect (more matched payments → better heuristics → more matching). Distribution: Streamflow/Sphere user bases as partners; the invoice-hash convention is published as an **open standard** other Solana payment tools can adopt. Skyfire ($9.5M raised) ceased operations in early 2026 chasing a proprietary protocol while the market standardized on open alternatives — the open-standard lesson is deliberate.

## competitiveLandscape

| Player | What they do | What they don't do |
|---|---|---|
| **Request Finance** | Full crypto AP/AR suite, QuickBooks/Xero/NetSuite sync, Solana USDC/USDT since 2024, $1.2B cumulative volume | EVM-origin, $300+/mo, no Solana-native protocol binding, no agent receipts |
| **Streamflow** | Solana payroll streaming + batch payouts, $1.6B TVL, 1.3M users | No invoicing, no reconciliation, no disputes |
| **MoonPay Commerce (ex-Helio)** | Merchant checkout, $1.5B processed, powers Shopify Solana Pay | Point-of-sale focus; leaves the books to the merchant |
| **x402 / Pay.sh / Circle Agent Stack** | Machine payment rails, nanopayments down to $0.000001 | No receipts, refunds, or disputes defined; roadmap empty |
| **GhostPay-B2B, StealthBooks** | Hackathon-grade B2B accounting tools on GitHub | Not production; confirms demand, no protocol binding |

**The wedge:** Token-2022 transfer hooks make invoice-binding a protocol property rather than a SaaS feature — EVM-origin competitors can't retrofit it without an indexer rewrite, and Solana payment incumbents would have to adopt the convention (open standard) or rebuild their stack. Nobody in the landscape owns settlement truth; Matchbook's only job is to own it.

---

## Research base (primary sources, gathered 2026-09-04)

- Solana stablecoin volume & senders: everstake.one (Artemis data); solana.com/docs/payments
- Reconciliation problem statement: solanacompass.com/projects/request-network, solanacompass.com/projects/request-finance
- Sablier Solana wind-down: blog.sablier.com/solana-shutdown; cryptobriefing.com
- Streamflow scale: crypto.news/streamflow-hits-1-6b-tvl; madeonsol.com/tools/streamflow
- x402 adoption, gaps, roadmap status: docs.x402.org; github.com/x402-foundation/x402; chainalysis.com/blog/x402-agentic-payments-adoption/; presenc.ai/research/x402-protocol-adoption-tracker-2026
- Pay.sh launch (Solana Foundation × Google Cloud): solana.com/news/pay-sh-launch
- Circle Agent Stack nanopayments: blockhead.co (May 12, 2026)
- Gusto USDC contractor payouts: gusto.com/company-news/same-day-international-contractor-payments
- Skyfire wind-down: archtools.dev/blog-skyfire-migration.html
- MoonPay/Helio: moonpay.com/newsroom/helio-acquisition; cointelegraph.com
- Request Finance × Aleo ZK payroll ($3.7M in weeks): solanacompass.com/projects/request-finance

*All figures as reported by the linked sources in 2025–2026 coverage; compiled 2026-09-04.*
---

**[Matchbook Labs catalog](https://jayjex.github.io/matchbook-labs/)** — one page linking every Matchbook Labs product, MCP server, and repo.

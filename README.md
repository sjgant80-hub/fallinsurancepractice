# FallInsurancePractice · v1.0.0

**Sovereign UK insurance broker firm-accounting.** Single HTML file. IDB primary. Zero servers. MIT.

Part of the **`fallinsurance` bundle** (anchor 829 · onboard 839 · paper 853 · **practice 857**) — fourth and final tool, sitting where the money does. Where `fallinsurance` quotes and binds, `fallinsuranceonboard` does KYC/AML, and `fallinsurancepaper` issues IPIDs and demands & needs statements, **`fallinsurancepractice` runs the till** — CASS 5 client money, commission ledger, IPT remittance, renewal pipeline P&L, PI/FCA/BIBA accruals.

---

## For brokers

You run a 1-10 person UK GI broker. Maybe DA, maybe AR under a Cobra / Bspoke / Movo / Bravo network. You handle commercial property, EL, PL, motor fleet, cyber, PI, D&O, landlord, sometimes personal lines. The FCA cares about three things you cannot get wrong: **client money (CASS 5), PI cover, conduct disclosure**. This tool covers the first two and feeds the third.

### The killer feature: CASS 5

Most accounting tools treat client money as "an account". CASS 5 treats it as a **statutory trust** — and FCA enforces against the trust. FallInsurancePractice gives you:

- **Separate `clientAccount` + `officeAccount` ledgers** — never mix
- **Per-policy balance** — see exactly how much of the trust money belongs to each policy
- **Monthly reconciliation workflow (CASS 5.5.63)** — adviser enters bank balance, tool computes discrepancy, sha256-stamps supporting docs, records adviser id + ts. Amber at 25d, red at 45d.
- **Mixed payment splits (CASS 5.3.5R)** — log the three same-day legs: gross-in to client account → net premium + IPT to insurer → commission + fee to office account. Tool warns if legs don't balance.
- **Premium-to-insurer remittance tracker** — per-policy ageing against your TOBA terms (default 30 days, configurable). Red if you're sitting on insurer money past terms.
- **Stale balance detection** — positive client-account balances held >180 days. Investigate, refund, or charity-donate with FCA notification.

### Commission ledger

- Per policy: gross premium, IPT (12% standard / 20% higher for travel + extended warranty), broker fee, commission %, commission £, net to insurer.
- Aggregations: by insurer, by adviser, by month, new business vs renewal split.
- AR network levy: configure the % retained by your network (typically 15-30%) to see net commission to the firm.

### IPT remittance tracker

You don't pay IPT — you collect it from the client, pass it to the insurer with the net premium, the insurer pays HMRC. But FCA, your insurer audit, and HMRC investigations all want a clean per-insurer-month reconciliation: **IPT collected vs IPT remitted vs balance**. That's a tab in this tool.

### Renewal pipeline P&L

Next 12 months of renewals bucketed by month, multiplied by your conversion rate (default 85%, configurable). Forecast commission vs prior-year actual. Per-policy 90-day queue colour-coded by urgency.

### Per-adviser P&L

Each adviser: total commission, YTD/MTD, policies + clients under advice, new biz vs renewal split, in-force conversion rate, average commission per case, % of firm commission. CPD hours tracked against the 15-hr FCA minimum.

### Firm P&L + accruals

Monthly commission - network levy (if AR) - recorded expenses - accruals (PI / FCA / BIBA / AML supervision) = net. MoM and YoY comparisons. Seed-monthly-accruals button drops the four standard accrual lines into the current month.

### PI insurance tracking

FCA SYSC minimum for insurance brokers: **£1.5M single claim, £2.25M aggregate** (higher than IFA minimums). Insurer / policy / cover / expiry / annual premium. Monthly accrual baked into firm P&L. Amber 60d, red 30d before expiry.

### FCA / BIBA / AML fees

Annual estimates that monthly-accrue into firm P&L. FCA fee block A.19 (general insurance distribution). BIBA membership. AML supervision (FCA is your AML supervisor as an FCA-regulated firm).

### 14 T0 rules · ask anything

Offline rules engine answers:
- CASS 5 reconciliation cadence
- Mixed payment rule (CASS 5.3.5R)
- Premium-to-insurer remittance timing
- IPT standard vs higher rate
- PI minimum cover (insurance broker)
- FCA fee bands
- Broker vs agent capacity (ICOBS 2.3.3R)
- Commission disclosure rules (ICOBS 4.4 + IDD)
- Net of commission vs gross premium
- Premium financing T&Cs (ICOBS 5 + CONC)
- Network AR vs DA levy
- Stale client money (CASS 5)
- Insurer audit / market account reconciliation
- ICOBS principles for client money

Add an API key (Anthropic / OpenAI / Gemini / OpenRouter) for T3 grounded answers off your firm's real numbers.

---

## For developers

### Architecture

- One `index.html` · ~137KB · 1928 lines · vanilla JS · zero deps
- IndexedDB primary (10 stores) · localStorage fallback for settings
- BroadcastChannel mesh: `fall-insurance` (bundle) + `fall-client` (IFA base) + `fall-signal` (estate-wide)
- P3 audit chain · sha256 prevHash linkage · 6-year FCA SYSC retention
- T0/T3 cascade (T0 rules engine offline · T3 BYOK LLM)
- PWA manifest via data: URL · installable

### IDB stores (10)

`state` · `clients` · `advisers` · `firms` · `policies` · `clientAccount` · `officeAccount` · `reconciliations` · `expenses` · `audit`

### Schema conformance

Conforms to `INSURANCE-BUNDLE-SHARED-SCHEMA.md` v1.0:
- `Policy` record (productClass, premium {gross,ipt,fee,commission,net}, demandsAndNeeds, cdSummary, midTermAdjustments, claims)
- `InsuranceClient` extends IFA base with `industry`, `companiesHouseNo`, `riskProfile`, `policiesHeld`
- `InsuranceAdviser` extends IFA base with `fcaRefNo`, `smcrRole`, `cpdHours`, `iddCompliance`
- BroadcastChannel `fall-insurance` messages: `policy.created/updated/lapsed/claimed/renewed`, `client.*`, `adviser.*`, `firm.*`, `commission.received`, `sync.request/snapshot`, `clientAccount.entry`, `reconciliation.done`
- Inherits IFA base `Client`/`Adviser`/`Firm` schema for cross-bundle interop

### Mesh integration

On boot: emits `sync.request` on both `fall-insurance` and `fall-client`. Any sibling tool with populated state responds with `sync.snapshot`. Receiver merges by `updatedAt` (later wins).

On commission receipt from a sibling (`commission.received`): auto-creates an `officeAccount` entry with `source: 'commission-in'` and an audit record. On any client/adviser/firm mutation: debounced (300ms) broadcast of the full record.

### Constants

- `TOOLNAME = 'fallinsurancepractice'`
- `VERSION = '1.0.0'`
- `PRIME = 857`
- Tax year `2025-26`
- CASS 5 amber 25d / red 45d
- PI min single £1.5M / aggregate £2.25M
- IPT standard 0.12 / higher 0.20
- Stale balance 180d

### Quick start

1. Open `index.html` in any modern browser. No build step.
2. Demo data seeds automatically on empty state (1 firm + 1 adviser + 1 client + 1 commercial-property policy + 3 CASS 5 client-account entries + 1 reconciliation + monthly accruals).
3. Open Settings to set your real firm name, FCA ref, PI cover, network config.
4. Use Quick Actions on the dashboard for record-fee, record-receipt, record-remittance, monthly-reconcile.

### Sovereignty

- All data lives in IndexedDB on the device. Nothing leaves unless you explicitly enter an API key for T3.
- Browser-side sha256 via `crypto.subtle` for the audit chain.
- Export everything as CSV (commission), markdown (firm P&L), or JSON (audit chain).
- "Wipe all data" in Settings is a real wipe — no remote backup, by design.

### Disclaimer

FallInsurancePractice is a tool for UK FCA-regulated insurance brokers. It assists with policy management, IDD-compliant demands & needs documentation, CASS 5 client money tracking, and commission accounting. It is not a quote/bind/issue platform — insurer integrations remain the broker's responsibility. Sovereign — client data never leaves the device.

Always verify with your compliance consultant before acting. This is research and bookkeeping assistance, not regulated advice.

---

## License

MIT — see `LICENSE`.

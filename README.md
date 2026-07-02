# Cixtant Public: Sovereign Friction Data Feed

Cixtant Public is a free, open-access, on-chain oracle data feed on Solana that publishes jurisdiction-level sovereign friction scores to protect capital deployment in underserved environments.

Cixtant implements the **Signal Economies Framework** (Gil, 2025) — a spectral theory of institutional friction that maps sovereign governance data into a four-dimensional state vector and computes a dominant eigenvalue that predicts capital formation capacity, investment risk, and policy leverage.

****Public API preview:** `https://cixtant.vercel.app/api/public/signal?jurisdiction=rwanda`

## Core Methodology & Schema
The data feed translates macro-economic risk dimensions into a mathematically derived friction score. The payload pushed on-chain includes:
* **Eigenvalue (λ₁):** The core institutional friction score.
* **Friction-Adjusted WACC:** Adjusted cost of capital metrics based on localized risk.
* **Risk Dimensions:** Segmented scores tracking Legal/Regulatory, Fiduciary, Value Appropriability, and Disclosure constraints.

* Every jurisdiction gets an **IFD State Vector** [D, F, V, L]:

| Dimension | Measures | Primary source |
|-----------|----------|----------------|
| **D** — Disclosure / Audit | Government effectiveness, corruption control | World Bank WGI |
| **F** — Fiduciary / Liability | Rule of law, legal rights, insolvency speed | World Bank WGI + WDI |
| **V** — Value Appropriability | Political stability, FDI inflows, voice | World Bank WGI |
| **L** — Legal / Regulatory | Regulatory quality, business start time | World Bank WGI + WDI |

The vector feeds a 4×4 **interaction matrix A** where off-diagonal terms encode path-dependence coupling between dimensions. The dominant eigenvalue λ₁ (via power iteration) and full complex spectrum (via numpy) determine the **eigenvalue regime**:

| λ₁ range | Regime | Meaning |
|----------|--------|---------|
| < 0.5 | Decoupled Convergence | Frictions are weak and self-correcting |
| 0.5 – 0.8 | Transitional Coupling | Frictions interact but haven't locked in |
| 0.8 – 1.0 | Entrenched Resonance | Frictions self-reinforce — reform is hard |
| ≥ 1.0 | Systemic Divergence | Frictions amplify — instability risk |

On top of λ₁, the platform classifies each jurisdiction by **institutional archetype** using cosine similarity to idealized [D,F,V,L] profiles, centered on the 0.5 neutral baseline so shape and magnitude are independent. A country scoring (0.9, 0.9, 0.9, 0.9) and one scoring (0.3, 0.3, 0.3, 0.3) point in opposite structural directions — Euclidean distance misses this; cosine similarity catches it.

## Output schema
## Public preview output

Available at `/api/public/signal?jurisdiction={key}` — no auth required:

```json
{
  "jurisdiction": { "name": "...", "iso2": "RW", "region": "Sub-Saharan Africa" },
  "ifd_state":    { "D": 0.52, "F": 0.48, "V": 0.41, "L": 0.50 },
  "_tier": "public",
  "jurisdiction": { "name": "Republic of Rwanda", "iso2": "RW", "region": "Sub-Saharan Africa" },
  "eigenvalue": {
    "regime":   "Entrenched Resonance",
    "lambda1":  0.883,
    "institutional_archetype": {
      "archetype":         "Expropriation Risk",
      "cosine_similarity": 0.812,
      "confidence":        "medium"
    },
    "full_spectrum": {
      "spectral_radius":   0.883,
      "has_complex_modes": false,
      "near_unity_count":  1,
      "dynamics_note":     "Monotone dynamics — frictions converge or diverge without oscillation"
    },
    "binding_constraint": "V"
    "regime": "Transitional Coupling",
    "lambda1": 0.680618,
    "binding_constraint": "D",
    "hydra_risk": false
  },
  "signal": { "alpha": 42.1, "f": 0.38, "classification": "MONITOR — balanced risk/reward" },
  "wacc":   { "wacc": 0.1432, "roic_wacc_ratio": 0.71 },
  "icv": {
    "friction_tax": 0.31,
    "hazard_rate":  0.371564,
    "interpretation": "Severe institutional discount"
  }
  "signal":  { "classification": "CAUTION — strong signal but value-destroying at current ROIC" },
  "phase":   { "classification": "IMPROVE" },
  "icv":     { "interpretation": "Material friction" }
}
```

---

## Jurisdictions covered

**26 jurisdictions** across four regions:

- **Latin America & Caribbean** — Brazil, Colombia, Chile, Mexico, Peru, Uruguay, Argentina, Puerto Rico
- **Sub-Saharan Africa** — Rwanda, Kenya, Ghana, Nigeria
- **Asia-Pacific (Developed)** — Japan, South Korea, Singapore
- **Asia-Pacific (Emerging)** — India, China, Indonesia, Malaysia, Thailand, Philippines, Vietnam
- **Asia-Pacific (Frontier)** — Bangladesh, Pakistan, Sri Lanka, Cambodia

---

## Architecture

```
World Bank WDI + WGI   ─┐
IMF WEO / IFS          ─┤─► Provider Registry ─► signal_resolver ─► SignalResolver ─► API
TI CPI + WJP fallbacks ─┘
```

- **Backend:** FastAPI + Mangum on Vercel serverless (Python)
- **Frontend:** Next.js 14 App Router, Supabase Auth
- **Data:** World Bank API v2 (WDI source=2, WGI source=3), parallel fetch via ThreadPoolExecutor
- **Fallbacks:** TI CPI 2023 + WJP Rule of Law 2023 embedded for governance gaps
- **Caching:** Vercel KV (Redis) with 24h TTL; in-memory fallback

---

## API endpoints

| Endpoint | Description |
|----------|------|-------------|
| `GET /api/public/signal?jurisdiction={key}` | None | Public preview — regime + classification |
| `GET /api/public/jurisdictions` | None | List all supported jurisdictions |
| `GET /api/signal?jurisdiction={key}` | Bearer token | Full premium assessment |
| `GET /api/signal/batch?jurisdictions=rwanda,brazil` | Bearer token | Multi-jurisdiction |
| `GET /api/monitoring/{jurisdiction}` | Bearer token | Monitoring metrics |
| `GET /api/docs` | None | Interactive API docs (Swagger) |

---

## Local development

```bash
# Frontend
npm install
npm run dev

# Backend (Python)
pip install -r requirements.txt
uvicorn api.index:app --reload
```

### Environment variables

Copy `.env.example` to `.env.local` and fill in your values:

```bash
cp .env.example .env.local
```

---

## Methodology & Academic Sourcing
The calculation engines driving the on-chain data schema are based on the Institutional Friction Decomposition (IFD) framework. 
For the underlying financial modeling, asset pricing structures, and mathematical proofs regarding how institutional uncertainty shifts discount rates, please see the foundational literature:
* **Paper Title:** *Institutional Friction Decomposition of Valuation Discounts: A Structured Diagnostic Framework for Identifying Institutional Sources of Persistent Discount Rate Distortions*
* **Author:** Aidah Gil (March 2026)
* **Access:** Available for review on [SSRN](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=6427278)

---

## Why Solana?

Existing Solana oracle infrastructure — Chainlink, Pyth, Switchboard, DIA, RedStone — answers one question: *what is this asset worth right now.* None of them answer a different one that matters just as much for cross-border and RWA-backed positions: *how much institutional friction does the underlying jurisdiction add to actually realizing that value.* Cixtant is not a price oracle — it's the first institutional/regulatory-risk oracle on Solana, designed to sit alongside price feeds, not replace them.

This matters more on Solana than elsewhere: independent analytics (Sentora, 2026) show 43.7% of Solana's RWA market cap is actively deployed as DeFi lending collateral, vs. 6.1% on Ethereum — Solana's RWA ecosystem is built around active financial use, not idle custody. A real-time friction score only earns its keep where protocols are making real-time decisions with it. That's Solana's current profile more than any other chain's.

Beyond the oracle layer:
- **Cost at scale:** quarterly updates across 26 jurisdictions cost fractions of a cent per cycle, keeping the on-chain feed sustainable as a free public good.
- **Micropayment-native monetization:** sub-cent, on-demand queries (e.g. $0.05 USDC per lookup) are commercially viable on Solana in a way fixed-fee card processors ($0.30 + 3%) can't support — a path to reach users who can't justify a subscription.
- **Reach into underserved markets:** Solana's low transaction cost makes the feed genuinely queryable from Cuba and other low-connectivity, low-income environments — where this kind of institutional intelligence is most needed and currently least available.

Could the on-chain feed run on another low-fee chain? Technically, yes — the architecture isn't Solana-exclusive. What's harder to replicate is the audience: Solana currently has the highest concentration of protocols treating RWAs as active collateral rather than static custody, which is exactly the behavior a real-time institutional-risk feed is built to serve.

---

## Repository Roadmap (Instagrant Milestone — Data Pipeline Hardening)
- [ ] Fix `population` field (add SP.POP.TOTL to fetch layer)
- [ ] Fix WGI endpoint routing (source=3, mrv=10, most-recent-non-null-year)
- [x] TI CPI + WJP fallback sources integrated for governance gaps
- [ ] Extend TI CPI + WJP fallback coverage to Asia-Pacific jurisdictions

---

## License

The on-chain program, SDK, and API interface are open source (MIT). The Signal Economies Framework's calibrated scoring engine (archetype vectors, interaction matrix weights) is proprietary.
Ingenio CIX retains all rights to the underlying methodology.

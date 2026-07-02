# Cixtant Public: Sovereign Friction Data Feed

Cixtant Public is a free, open-access, on-chain oracle data feed on Solana that publishes jurisdiction-level sovereign friction scores to protect capital deployment in underserved environments.

Cixtant implements the **Signal Economies Framework** (Gil, 2025) — a spectral theory of institutional friction that maps sovereign governance data into a four-dimensional state vector and computes a dominant eigenvalue that predicts capital formation capacity, investment risk, and policy leverage.

**Live API:** `https://cixtant.vercel.app/api/signal?jurisdiction=rwanda`

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

```json
{
  "jurisdiction": { "name": "...", "iso2": "RW", "region": "Sub-Saharan Africa" },
  "ifd_state":    { "D": 0.52, "F": 0.48, "V": 0.41, "L": 0.50 },
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
  },
  "signal": { "alpha": 42.1, "f": 0.38, "classification": "MONITOR — balanced risk/reward" },
  "wacc":   { "wacc": 0.1432, "roic_wacc_ratio": 0.71 },
  "icv": {
    "friction_tax": 0.31,
    "hazard_rate":  0.371564,
    "interpretation": "Severe institutional discount"
  }
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
|----------|-------------|
| `GET /api/signal?jurisdiction={key}` | Full sovereign assessment |
| `GET /api/signal/batch?jurisdictions=rwanda,brazil` | Multi-jurisdiction |
| `GET /api/signal/jurisdictions` | List all supported jurisdictions |
| `GET /api/monitoring/{jurisdiction}` | Monitoring metrics |
| `GET /api/docs` | Interactive API docs (Swagger) |

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

## Why Solana?
Cixtant Public is specifically engineered to serve user bases in Cuba and other heavily sanctioned, low-connectivity emerging markets. 
* **Ultra-Low Cost:** Solana's near-zero transaction fees make querying state data realistic for low-income entrepreneurs.
* **High Efficiency:** Low-bandwidth, high-latency user environments require rapid processing speeds to safely hedge wealth against sudden domestic fiat currency debasement.

## Repository Roadmap (Instagrant Milestone)
- [ ] Initialize Anchor program framework.
- [ ] Define on-chain Account state serialization for eigenvalue payloads.
- [ ] Build the off-chain data ingestion script to push metrics to Solana Devnet.


## Roadmap

- [ ] `/public` demo endpoint (no auth required)
- [ ] TI CPI + WJP data for Asia-Pacific jurisdictions
- [ ] Point-in-time backtesting (`as_of_date` parameter)
- [ ] Annual governance data refresh pipeline
- [ ] Solana on-chain signal anchoring

---

## License

Proprietary — Ingenio CIX / Cixtant. The Signal Economies Framework and IFD methodology are original research. Contact [ingeniocix](https://github.com/IngenioCIX) for licensing inquiries.

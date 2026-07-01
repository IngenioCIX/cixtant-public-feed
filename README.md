# Cixtant Public: Sovereign Friction Data Feed

Cixtant Public is a free, open-access, on-chain oracle data feed on Solana that publishes jurisdiction-level sovereign friction scores to protect capital deployment in underserved environments.

## Core Methodology & Schema
The data feed translates macro-economic risk dimensions into a mathematically derived friction score. The payload pushed on-chain includes:
* **Eigenvalue (λ₁):** The core institutional friction score.
* **Friction-Adjusted WACC:** Adjusted cost of capital metrics based on localized risk.
* **Risk Dimensions:** Segmented scores tracking Legal/Regulatory, Fiduciary, Value Appropriability, and Disclosure constraints.

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

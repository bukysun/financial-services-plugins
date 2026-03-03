---
name: bond-futures-basis
description: Analyze the bond futures basis by pricing futures, identifying the cheapest-to-deliver, and comparing with yield curves to assess delivery option value and basis trading opportunities. Use when analyzing bond futures, computing the basis, identifying CTD bonds, calculating implied repo rates, or evaluating basis trades.
---

# Bond Futures Basis Analysis

You are an expert in bond futures and basis trading. Combine futures pricing, cash bond analytics, yield curve data, and historical tracking to assess basis trade opportunities. Focus on routing data from MCP tools into a coherent basis analysis — let the tools compute, you interpret and present.

## Core Principles

The basis sits at the intersection of cash bond pricing, repo markets, and delivery mechanics. Always start by pricing the future to identify the CTD and delivery basket, then price the CTD bond separately, compute basis metrics from the two outputs, and overlay yield curve context. The net basis represents embedded delivery option value — compare implied repo to market repo to assess whether futures are rich or cheap.

## Available MCP Tools

- **FRED MCP (`fred_get_series`)** — Treasury yields for CTD bond yield approximation and implied repo calculation. Key series: DGS2, DGS5, DGS10, DGS30.
- **FRED MCP (`fred_search`)** — Search for repo rate series (e.g., SOFR, EFFR) for financing cost baseline.
- **Yahoo Finance MCP (`get_quote`)** — Bond futures quotes where available on Yahoo Finance (e.g., "ZN=F" for 10Y T-note futures, "ZB=F" for 30Y T-bond futures).
- **Yahoo Finance MCP (`get_historical`)** — Historical futures price data for basis trend analysis.

> **Note on CTD identification:** Identifying the cheapest-to-deliver (CTD) bond requires live bond pricing data (ISIN/CUSIP prices) which is not available from free sources. CTD identification must rely on web search for current CME delivery basket data or dealer research.

## Tool Chaining Workflow

1. **Futures Quote:** Call `get_quote` on Yahoo Finance for the futures contract (e.g., "ZN=F" for 10Y T-note futures). Get current futures price and contract specs.
2. **CTD Identification (web search):** Search web for "[futures contract] CTD cheapest-to-deliver bond [current month]". Note the CTD CUSIP, coupon, maturity, and conversion factor.
3. **Treasury Yield Baseline:** Call `fred_get_series` for the yield closest to the CTD maturity (DGS5, DGS10, etc.). Get current yield for basis calculation.
4. **Repo Rate:** Call `fred_get_series` for SOFR (overnight repo proxy) and EFFR. Use the most relevant short-term rate as the financing cost.
5. **Basis Calculation:** Basis (gross) = CTD price − (Futures price × Conversion factor). Implied repo rate = [(Forward price / Spot price) − 1] × (360 / days to delivery). Compare implied repo to SOFR to determine if basis is rich or cheap.
6. **Historical Context:** Call `get_historical` for the futures contract to assess where the basis sits vs recent history.
7. **Synthesize:** Present basis level, implied repo vs actual repo spread, and CTD analysis. Note data quality limitations from using approximated prices.

## Output Format

### Future Summary
| Field | Value |
|-------|-------|
| Contract | ... |
| Fair Price | ... |
| CTD Bond | ... |
| Conversion Factor | ... |
| Contract DV01 | ... |

### CTD Bond Analytics
| Field | Value |
|-------|-------|
| Clean Price | ... |
| YTM | ... |
| Duration | ... |
| DV01 | ... |

### Basis Calculation
| Metric | Value |
|--------|-------|
| Gross Basis | ... ticks |
| Carry | ... ticks |
| Net Basis | ... ticks |
| Implied Repo | ...% |
| Market Repo (approx) | ...% |
| Assessment | Rich / Fair / Cheap |

### Historical Basis Context
| Metric | Current | 3M Avg | 6M Avg | Percentile |
|--------|---------|--------|--------|------------|
| Net Basis | ... | ... | ... | ...th |
| Implied Repo | ... | ... | ... | ...th |

Lead with the basis trade assessment (long/short/neutral) and implied repo comparison. Follow with detailed analytics tables.

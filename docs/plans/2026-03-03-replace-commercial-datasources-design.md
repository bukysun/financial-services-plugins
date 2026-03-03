# Design: Replace Commercial Data Sources with Free/Open-Source Alternatives

**Date:** 2026-03-03
**Status:** Approved
**Scope:** All plugins including partner-built (LSEG, S&P Global)

---

## Background

The financial-services-plugins repository currently depends on 11 commercial MCP data providers, all of which require paid subscriptions. The goal is to replace all commercial data sources with free or open-source alternatives while keeping functionality as close to parity as possible.

---

## MCP Replacement Mapping

### New Free MCP Servers (replacing 11 commercial servers)

| New Free MCP | Replaces | Capabilities |
|---|---|---|
| **Yahoo Finance MCP** | Daloopa, S&P Kensho, FactSet, Morningstar | Stock prices, fundamentals, financials, analyst estimates, ETF/fund data, options chains |
| **FRED MCP** | LSEG (macro/rates portion) | Interest rate curves, macroeconomic indicators, some FX rates |
| **SEC EDGAR MCP** | Daloopa (supplemental), LSEG (company fundamentals) | US public company filings, 10-K/10-Q, historical financial data |
| **Web Search (degraded fallback)** | Moody's, MT Newswires, Aiera, PitchBook, Chronograph | Credit ratings, financial news, earnings transcripts, PE/VC data |
| **Removed** | Egnyte | Document management (non-core) |

### MCP Server Configuration

**New `financial-analysis/.mcp.json`:**
```json
{
  "mcpServers": {
    "yahoo-finance": { "type": "http", "url": "<yahoo-finance-mcp-endpoint>" },
    "fred": { "type": "http", "url": "<fred-mcp-endpoint>" },
    "edgar": { "type": "http", "url": "<edgar-mcp-endpoint>" }
  }
}
```

**New `partner-built/lseg/.mcp.json`:**
```json
{
  "mcpServers": {
    "fred": { "type": "http", "url": "<fred-mcp-endpoint>" },
    "yahoo-finance": { "type": "http", "url": "<yahoo-finance-mcp-endpoint>" }
  }
}
```

**New `partner-built/spglobal/.mcp.json`:**
```json
{
  "mcpServers": {
    "yahoo-finance": { "type": "http", "url": "<yahoo-finance-mcp-endpoint>" },
    "edgar": { "type": "http", "url": "<edgar-mcp-endpoint>" }
  }
}
```

---

## Files to Modify (14 total)

### 1. MCP Configuration Files (3)
- `financial-analysis/.mcp.json` — Replace 11 commercial MCPs with 3 free MCPs
- `partner-built/lseg/.mcp.json` — LSEG → FRED + Yahoo Finance
- `partner-built/spglobal/.mcp.json` — S&P Global → Yahoo Finance + SEC EDGAR

### 2. Core Plugin Skills (1)
- `financial-analysis/skills/comps-analysis/SKILL.md`
  - Lines 28–33: Update data source priority from "S&P Kensho MCP, FactSet MCP, Daloopa MCP" → "Yahoo Finance MCP, SEC EDGAR MCP"
  - Line 239: Update "Data Sources & Quality" section
  - Line 410: Update "Gather data" step

### 3. LSEG Partner-Built Skills (8)
Each SKILL.md has instructions to call specific LSEG MCP tools. Update to call FRED/Yahoo Finance equivalent tools:
- `skills/bond-relative-value/SKILL.md` — LSEG bond tools → FRED rates + Yahoo Finance + web search
- `skills/fx-carry-trade/SKILL.md` — LSEG FX tools → FRED FX rates + Yahoo Finance
- `skills/equity-research/SKILL.md` — LSEG quant analytics → Yahoo Finance consensus + SEC EDGAR
- `skills/swap-curve-strategy/SKILL.md` — LSEG curves → FRED interest rate curves
- `skills/option-vol-analysis/SKILL.md` — LSEG vol surface → Yahoo Finance options chains
- `skills/fixed-income-portfolio/SKILL.md` — LSEG YieldBook → FRED rates + SEC EDGAR
- `skills/macro-rates-monitor/SKILL.md` — LSEG macro → FRED macro indicators
- `skills/bond-futures-basis/SKILL.md` — LSEG bond futures → FRED + Yahoo Finance + web search

### 4. S&P Global Partner-Built Skills (3)
- `skills/tear-sheet/SKILL.md` — Capital IQ data → Yahoo Finance + SEC EDGAR
- `skills/earnings-preview-beta/SKILL.md` — Capital IQ consensus → Yahoo Finance consensus estimates
- `skills/funding-digest/SKILL.md` — S&P data → SEC EDGAR + web search

### 5. Documentation Files (3)
- `partner-built/lseg/README.md` — Remove "requires LSEG credentials" requirement
- `partner-built/spglobal/README.md` — Remove paid subscription requirements
- `README.md` — Update MCP integrations table (11 providers → 3 providers)

---

## Capability Gap Analysis

| Original Feature | Replacement | Gap |
|---|---|---|
| Bond/options pricing (LSEG) | FRED rates + Yahoo Finance options chains | Cannot do OAS/Z-spread/institutional pricing |
| Credit ratings (Moody's) | Web search | Timeliness and completeness reduced |
| PE/VC data (PitchBook) | Web search | Private market data coverage limited |
| Real-time financial news (MT Newswires) | Web search | No structured news feed |
| Earnings call transcripts (Aiera) | Web search | No audio analysis |
| Document management (Egnyte) | Removed | Non-core, no replacement needed |

For features with capability gaps, skills will be updated to:
1. First try the free MCP equivalent
2. Fall back to web search with clear instructions
3. Note data quality limitations where relevant

---

## Free Data Source Notes

- **Yahoo Finance**: No API key required for basic use; rate limits apply
- **FRED**: Free API key required (register at fred.stlouisfed.org); generous rate limits
- **SEC EDGAR**: No authentication required for public company data
- **Web search**: Available in Claude natively; used as fallback only

---

## Out of Scope

- Replacing partner plugin author metadata (plugin.json author fields remain unchanged)
- Adding new financial workflows not present in the original plugins
- Implementing custom MCP servers (use existing community MCP servers)

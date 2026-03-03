# Replace Commercial Data Sources with Free Alternatives — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace all 11 commercial MCP data providers with 3 free/open-source MCPs (Yahoo Finance, FRED, SEC EDGAR) plus web search fallback, across all plugins including partner-built LSEG and S&P Global.

**Architecture:** Swap MCP configurations from paid HTTP endpoints to free stdio MCP servers. Update every skill file that references specific commercial tool names or data source priority instructions. Partner-built plugins are rewritten to use the same free MCPs as the core plugin.

**Tech Stack:** Markdown (skill files), JSON (MCP configs). No code compilation needed — edits take effect immediately.

---

## Task 1: Research free MCP server packages

**Files:**
- No files to edit — research only

**Step 1: Find Yahoo Finance MCP package**

Search npm for a maintained Yahoo Finance MCP server:
```bash
npm search mcp yahoo finance 2>/dev/null | head -20
```

Look for packages with recent updates and >100 weekly downloads. Good candidates:
- `mcp-server-yahoo-finance`
- `@quackquack/mcp-yahoo-finance`
- `mcp-yahoo`

**Step 2: Find FRED MCP package**

```bash
npm search mcp fred federal reserve 2>/dev/null | head -20
```

FRED API key is free at https://fred.stlouisfed.org/docs/api/api_key.html — register before implementation.

**Step 3: Find SEC EDGAR MCP package**

```bash
npm search mcp edgar sec 2>/dev/null | head -20
```

Also check: SEC EDGAR public API requires no auth key at https://data.sec.gov/

**Step 4: Confirm package names**

For each package found, verify it works:
```bash
npx -y <package-name> --help 2>&1 | head -5
```

**Step 5: Record the 3 confirmed package names**

You'll use these in Tasks 2 and 4–9. Format for .mcp.json will be:
```json
{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "<package-name>"],
  "env": { "API_KEY": "<key-if-needed>" }
}
```

**Step 6: Commit research notes**

```bash
git add -A
git commit -m "chore: document free MCP server package choices"
```

---

## Task 2: Replace financial-analysis/.mcp.json

**Files:**
- Modify: `financial-analysis/.mcp.json`

**Step 1: Verify current content**

Read `financial-analysis/.mcp.json` to confirm the 11 commercial entries.

**Step 2: Replace the file**

Replace the entire file with the 3 free MCPs. Use the package names confirmed in Task 1. Example structure (fill in actual package names from Task 1):

```json
{
  "mcpServers": {
    "yahoo-finance": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "<yahoo-finance-mcp-package>"]
    },
    "fred": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "<fred-mcp-package>"],
      "env": {
        "FRED_API_KEY": "<your-free-fred-api-key>"
      }
    },
    "edgar": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "<edgar-mcp-package>"]
    }
  }
}
```

**Step 3: Verify JSON is valid**

```bash
python3 -m json.tool financial-analysis/.mcp.json
```
Expected: prints formatted JSON with no errors.

**Step 4: Commit**

```bash
git add financial-analysis/.mcp.json
git commit -m "feat: replace 11 commercial MCPs with yahoo-finance, fred, edgar"
```

---

## Task 3: Update comps-analysis/SKILL.md data source references

**Files:**
- Modify: `financial-analysis/skills/comps-analysis/SKILL.md` (lines 28–33, 239, 410)

**Step 1: Read the file**

Read `financial-analysis/skills/comps-analysis/SKILL.md` to confirm line numbers.

**Step 2: Replace the critical data source priority block (lines 28–33)**

Find and replace:
```
**ALWAYS follow this data source hierarchy:**

1. **FIRST: Check for MCP data sources** - If S&P Kensho MCP, FactSet MCP, or Daloopa MCP are available, use them exclusively for financial and trading information
2. **DO NOT use web search** if the above MCP data sources are available
3. **ONLY if MCPs are unavailable:** Then use Bloomberg Terminal, SEC EDGAR filings, or other institutional sources
4. **NEVER use web search as a primary data source** - it lacks the accuracy, audit trails, and reliability required for institutional-grade analysis
```

Replace with:
```
**ALWAYS follow this data source hierarchy:**

1. **FIRST: Check for MCP data sources** - If Yahoo Finance MCP or SEC EDGAR MCP are available, use them for financial and trading information
2. **DO NOT use web search** if the above MCP data sources are available
3. **ONLY if MCPs are unavailable:** Then use SEC EDGAR filings directly (https://www.sec.gov/cgi-bin/browse-edgar) or web search
4. **Web search is a last resort** - prefer MCP sources for accuracy and audit trails
```

**Step 3: Update the "Data Sources & Quality" reference (line 239)**

Find:
```
- Where did the data come from? (S&P Kensho MCP, FactSet MCP, Daloopa MCP, Bloomberg, SEC filings)
```

Replace with:
```
- Where did the data come from? (Yahoo Finance MCP, SEC EDGAR MCP, SEC filings)
```

**Step 4: Update the "Gather data" workflow step (line 410)**

Find:
```
   - Pull from primary sources (S&P Kensho MCP, FactSet MCP, Daloopa MCP if available; otherwise Bloomberg, SEC)
```

Replace with:
```
   - Pull from primary sources (Yahoo Finance MCP, SEC EDGAR MCP if available; otherwise SEC EDGAR directly)
```

**Step 5: Commit**

```bash
git add financial-analysis/skills/comps-analysis/SKILL.md
git commit -m "feat: update comps-analysis to use yahoo-finance and edgar MCPs"
```

---

## Task 4: Replace partner-built/lseg/.mcp.json

**Files:**
- Modify: `partner-built/lseg/.mcp.json`

**Step 1: Read the file to confirm current content**

Read `partner-built/lseg/.mcp.json`.

**Step 2: Replace the file**

```json
{
  "mcpServers": {
    "fred": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "<fred-mcp-package>"],
      "env": {
        "FRED_API_KEY": "<your-free-fred-api-key>"
      }
    },
    "yahoo-finance": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "<yahoo-finance-mcp-package>"]
    }
  }
}
```

**Step 3: Validate JSON**

```bash
python3 -m json.tool partner-built/lseg/.mcp.json
```

**Step 4: Commit**

```bash
git add partner-built/lseg/.mcp.json
git commit -m "feat: replace LSEG MCP with fred and yahoo-finance"
```

---

## Task 5: Update LSEG equity-research/SKILL.md

**Files:**
- Modify: `partner-built/lseg/skills/equity-research/SKILL.md`

**Step 1: Read the file**

Read `partner-built/lseg/skills/equity-research/SKILL.md`.

**Step 2: Replace the "Available MCP Tools" section**

Find and replace the entire "## Available MCP Tools" block:

```markdown
## Available MCP Tools
```
↓ Replace with:
```markdown
## Available MCP Tools

- **Yahoo Finance MCP** — Stock quotes, historical prices (OHLCV), analyst consensus estimates (EPS, Revenue), company fundamentals (income statement, balance sheet, cash flow), dividends, beta. Use for all equity and company financial data.
- **FRED MCP** — Macroeconomic indicators (GDP, CPI, unemployment, PMI, PCE). Use to establish the economic backdrop for the company's sector. Search for series by keyword or FRED series ID.
```

**Step 3: Replace the "Tool Chaining Workflow" section**

Find and replace the entire "## Tool Chaining Workflow" block:

```markdown
## Tool Chaining Workflow

1. **Consensus Snapshot:** Use Yahoo Finance MCP to get analyst estimates for FY1 and FY2 (EPS, Revenue). Note analyst count and price targets.
2. **Historical Fundamentals:** Use Yahoo Finance MCP to fetch income statement, balance sheet, and cash flow for the last 3-5 fiscal years. Extract revenue growth, margins, leverage, returns (ROE, ROIC).
3. **Price Performance:** Use Yahoo Finance MCP to get 1-year price history. Compute YTD return, 1Y return, 52-week range position, beta.
4. **Macro Context:** Use FRED MCP to fetch GDP, CPI, and policy rate for the company's primary market. Search FRED for relevant series (e.g., "GDPC1" for US real GDP, "CPIAUCSL" for US CPI). Summarize whether macro is tailwind or headwind.
5. **Synthesize:** Combine into a research note with consensus tables, financials summary, valuation metrics (forward P/E from price / consensus EPS), and macro backdrop.
```

**Step 4: Commit**

```bash
git add partner-built/lseg/skills/equity-research/SKILL.md
git commit -m "feat: update lseg equity-research skill to use yahoo-finance and fred"
```

---

## Task 6: Update LSEG macro-rates-monitor/SKILL.md

**Files:**
- Modify: `partner-built/lseg/skills/macro-rates-monitor/SKILL.md`

**Step 1: Read the file**

**Step 2: Replace "Available MCP Tools"**

Replace the entire block:
```markdown
## Available MCP Tools

- **FRED MCP** — The primary source for all macroeconomic data and interest rate series. Covers GDP, CPI, PCE, unemployment, payrolls, PMI, retail sales (US and international), government bond yields at standard tenors (3M, 2Y, 5Y, 10Y, 30Y), TIPS yields (real rates), and breakeven inflation rates. Search by series ID (e.g., "DGS10" for 10Y Treasury yield, "DFII10" for 10Y TIPS, "T5YIE" for 5Y breakeven).
- **Yahoo Finance MCP** — Supplemental source for equity market conditions and FX rates if needed for financial conditions context.
```

**Step 3: Replace "Tool Chaining Workflow"**

Replace the entire block:
```markdown
## Tool Chaining Workflow

1. **Pull Macro Indicators:** Use FRED MCP to fetch GDP (GDPC1), CPI (CPIAUCSL), PCE (PCEPI), unemployment (UNRATE), and PMI (MANEMP or ISM data) for the US. For other countries, search FRED for equivalent series. Retrieve latest value and 12 months of history.
2. **Yield Curve Snapshot:** Use FRED MCP to fetch Treasury yields at standard tenors: 3M (DTB3), 2Y (DGS2), 5Y (DGS5), 10Y (DGS10), 30Y (DGS30). Compute 2s10s slope (DGS10 minus DGS2) and 3M-10Y slope. Classify curve shape (normal / flat / inverted).
3. **Real Rate Decomposition:** Use FRED MCP to fetch TIPS yields (DFII5, DFII10) and inflation breakevens (T5YIE for 5Y, T10YIE for 10Y). Real rate = TIPS yield. Breakeven = nominal yield minus TIPS yield. Assess whether real rates are accommodative (negative real) or restrictive (positive real).
4. **Swap Spread Context:** Swap spread data is not available from FRED. Note that swap spreads are unavailable and provide qualitative commentary on financial conditions based on credit spreads (ICE BofA OAS series on FRED if available) or web search for current conditions.
5. **Historical Context:** Use FRED MCP to fetch the 10Y Treasury yield (DGS10) history over 1–3 years. Assess where current yields sit vs recent history.
6. **Synthesize:** Combine into a dashboard: cycle position, curve signals, real rate regime, and overall macro assessment.
```

**Step 4: Add a note about limitations**

At the bottom of the file, append:

```markdown
## Data Availability Notes

- **Swap rates and credit curves:** Not available from FRED. Use web search for current swap rate quotes or dealer pricing pages.
- **FX vol surfaces:** Not available for free. Use Yahoo Finance for spot FX rates; for vol surfaces, note "data unavailable from free sources."
- **Non-US yield curves:** FRED covers some international rates (e.g., "IRLTLT01DEA156N" for German 10Y). Search FRED for country-specific series.
```

**Step 5: Commit**

```bash
git add partner-built/lseg/skills/macro-rates-monitor/SKILL.md
git commit -m "feat: update lseg macro-rates skill to use FRED"
```

---

## Task 7: Update LSEG bond-relative-value/SKILL.md

**Files:**
- Modify: `partner-built/lseg/skills/bond-relative-value/SKILL.md`

**Step 1: Read the file**

**Step 2: Replace "Available MCP Tools"**

```markdown
## Available MCP Tools

- **FRED MCP** — Government bond yields at standard tenors for G-spread calculation (DGS2, DGS5, DGS10, DGS30 for US Treasuries; equivalent series for other markets). Historical yield series for spread context. Corporate bond spread indices (ICE BofA series) as a proxy for credit curves.
- **Yahoo Finance MCP** — Bond quotes for some corporate bonds (where available). Use for historical price context.

> **Note on limitations:** Institutional-grade bond pricing tools (clean/dirty price from ISIN, OAS, Z-spread, YieldBook scenarios) are not available from free sources. The analysis below uses yield-based approximations instead.
```

**Step 3: Replace "Tool Chaining Workflow"**

```markdown
## Tool Chaining Workflow

1. **Get Risk-Free Curve:** Use FRED MCP to fetch Treasury yields at the bond's maturity tenor (interpolate between DGS2, DGS5, DGS10, DGS30). This is the risk-free rate for G-spread calculation.
2. **Estimate Credit Spread:** Use FRED MCP to fetch the ICE BofA credit spread index for the bond's rating category (e.g., "BAMLC0A4CBBB" for BBB corporate OAS). This is a proxy for the credit curve at the issuer's rating.
3. **Estimate G-Spread:** If the bond's current yield is available (from Yahoo Finance or web search), compute: G-spread = bond yield minus Treasury yield at matching maturity.
4. **Historical Context:** Use FRED MCP to fetch the relevant credit spread index history (1-3 years). Assess where current spreads sit vs history (percentile rank).
5. **Rate Scenario Analysis:** Using the current FRED Treasury curve, manually compute price sensitivity: for a ±100bp parallel shift, use modified duration approximation: ΔP ≈ -Duration × ΔY × Price.
6. **Synthesize:** Combine estimated spread decomposition and rate sensitivity into a rich/cheap assessment. Note that without live ISIN pricing, estimates are approximate.
```

**Step 4: Commit**

```bash
git add partner-built/lseg/skills/bond-relative-value/SKILL.md
git commit -m "feat: update lseg bond-rv skill to use FRED with limitations noted"
```

---

## Task 8: Update remaining 5 LSEG skills

**Files:**
- Modify: `partner-built/lseg/skills/fx-carry-trade/SKILL.md`
- Modify: `partner-built/lseg/skills/swap-curve-strategy/SKILL.md`
- Modify: `partner-built/lseg/skills/option-vol-analysis/SKILL.md`
- Modify: `partner-built/lseg/skills/fixed-income-portfolio/SKILL.md`
- Modify: `partner-built/lseg/skills/bond-futures-basis/SKILL.md`

Read each file, then apply the same pattern as Tasks 5–7: replace the "Available MCP Tools" and "Tool Chaining Workflow" sections. Specific replacements:

**fx-carry-trade/SKILL.md:**
- `fx_spot_price`, `fx_forward_price`, `fx_forward_curve` → FRED FX series (e.g., "DEXUSEU" for EUR/USD) + Yahoo Finance FX for spot
- `fx_vol_surface` → "Not available from free sources. Use web search for broker-quoted vol or note data unavailable."
- `interest_rate_curve` → FRED yield curves as in Task 6

**swap-curve-strategy/SKILL.md:**
- All `interest_rate_curve`, `inflation_curve`, `ir_swap` tools → FRED equivalents (DGS series, TIPS series, FRED SOFR rates)
- Note swap rates not available; use SOFR term rates from FRED as proxy

**option-vol-analysis/SKILL.md:**
- `option_value` with Greeks → Yahoo Finance MCP options chain (provides strikes, expiries, implied vol, market prices)
- For Greeks: compute analytically from the option chain data (delta approximation from call/put price symmetry)
- `fx_vol_surface` → Not available; note limitation

**fixed-income-portfolio/SKILL.md:**
- `yieldbook_*` tools → FRED for yield/duration inputs; note that full portfolio analytics (OAS, cashflow present value) require manual approximation using modified duration
- Use Yahoo Finance for equity pricing components if mixed portfolio

**bond-futures-basis/SKILL.md:**
- `bond_price` for CTD → Web search for CME bond futures contract specs + FRED Treasury yields
- Note: Implied repo rate calculation requires live bond pricing; approximate using FRED data

For each file, after editing:
```bash
git add partner-built/lseg/skills/<skill-name>/SKILL.md
git commit -m "feat: update lseg <skill-name> skill to use free data sources"
```

---

## Task 9: Replace partner-built/spglobal/.mcp.json

**Files:**
- Modify: `partner-built/spglobal/.mcp.json`

**Step 1: Read the file**

**Step 2: Replace**

```json
{
  "mcpServers": {
    "yahoo-finance": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "<yahoo-finance-mcp-package>"]
    },
    "edgar": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "<edgar-mcp-package>"]
    }
  }
}
```

**Step 3: Validate**

```bash
python3 -m json.tool partner-built/spglobal/.mcp.json
```

**Step 4: Commit**

```bash
git add partner-built/spglobal/.mcp.json
git commit -m "feat: replace S&P Global MCP with yahoo-finance and edgar"
```

---

## Task 10: Update spglobal tear-sheet/SKILL.md

This is the most involved change. The skill has S&P Capital IQ hardcoded in multiple places.

**Files:**
- Modify: `partner-built/spglobal/skills/tear-sheet/SKILL.md`

**Step 1: Read the file to confirm all occurrences**

**Step 2: Replace the "Pull Data" step (Step 3)**

Find the block starting with:
```
Use the **S&P Global** MCP tools (also known as the Kensho LLM-ready API).
```

Replace with:
```
Use **Yahoo Finance MCP** for market data, fundamentals, analyst estimates, earnings history, and options. Use **SEC EDGAR MCP** for official filing data (10-K, 10-Q, 8-K), segment disclosures, and management discussion sections. The query plans in each reference file describe what data to retrieve — map these to Yahoo Finance and SEC EDGAR tools.

**Private company handling:** Yahoo Finance and SEC EDGAR cover US public companies only. For private companies, skip: stock price, 52-week range, beta, consensus estimates, trading comps. Use web search for any available private company data and note "Limited data — private company" prominently.
```

**Step 3: Update Data Integrity Rule 1**

Find:
```
1. **S&P Global tools are the only source for financial data.** Do not fill gaps with training knowledge — it may be stale or wrong.
```

Replace with:
```
1. **Yahoo Finance MCP and SEC EDGAR MCP are the primary sources for financial data.** Do not fill gaps with training knowledge — it may be stale or wrong. Web search is permitted as a fallback for data not available in these MCPs, but must be labeled as "Source: Web search."
```

**Step 4: Update Data Integrity Rule 10**

Find:
```
10. **No S&P Global tool returns executive or management data.**
```

Replace with:
```
10. **Yahoo Finance and SEC EDGAR may return executive data from filings, but verify it is current** — proxy statement data can be months old. If management names are returned, include the "as of" date of the source filing.
```

**Step 5: Update the footer in createFooter function**

Find:
```
text: `Data: S&P Capital IQ via Kensho | Analysis: AI-generated | ${date}`,
```

Replace with:
```
text: `Data: Yahoo Finance & SEC EDGAR | Analysis: AI-generated | ${date}`,
```

**Step 6: Update the footer documentation**

Find:
```
- Line 1: "Data: S&P Capital IQ via Kensho | Analysis: AI-generated | [Month Day, Year]"
```

Replace with:
```
- Line 1: "Data: Yahoo Finance & SEC EDGAR | Analysis: AI-generated | [Month Day, Year]"
```

**Step 7: Update the description frontmatter**

Find:
```
description: "Generate professional company tear sheets using S&P Capital IQ data via the Kensho LLM-ready API MCP server.
```

Replace with:
```
description: "Generate professional company tear sheets using Yahoo Finance and SEC EDGAR data.
```

**Step 8: Verify no remaining S&P/Kensho references**

```bash
grep -n "S&P\|Kensho\|Capital IQ\|kfinance" partner-built/spglobal/skills/tear-sheet/SKILL.md
```

Expected: zero matches (or only in commented-out historical notes). If any remain, fix them.

**Step 9: Commit**

```bash
git add partner-built/spglobal/skills/tear-sheet/SKILL.md
git commit -m "feat: update tear-sheet skill to use yahoo-finance and edgar"
```

---

## Task 11: Update spglobal earnings-preview-beta/SKILL.md

**Files:**
- Modify: `partner-built/spglobal/skills/earnings-preview-beta/SKILL.md`

**Step 1: Read the file**

**Step 2: Replace the data sources rule (top of file)**

Find:
```
**Data Sources (ZERO EXCEPTIONS):** The ONLY permitted data sources are **Kensho Grounding MCP** (`search`) and **S&P Global MCP** (`kfinance`). Absolutely NO other tools...
```

Replace with:
```
**Data Sources:** The primary data sources are **Yahoo Finance MCP** and **SEC EDGAR MCP**. Web search is permitted for news, analyst ratings, and any data not available from these MCPs. Specifically:
- Use **Yahoo Finance MCP** for: stock prices, EPS history, consensus estimates, market cap, competitor list, earnings dates, segments.
- Use **SEC EDGAR MCP** for: official filing-based financials, segment disclosures, 10-Q/10-K data.
- Use **web search** for: recent news, analyst commentary, earnings call transcripts (when not available via MCP).
- Do NOT use training knowledge for financial figures — it may be stale.
```

**Step 3: Replace all `kfinance` tool references**

For each function call in Phase 1–5, replace with Yahoo Finance MCP equivalents:

| Original (kfinance) | Replacement (Yahoo Finance MCP) |
|---|---|
| `get_info_from_identifiers` | Yahoo Finance company info tool |
| `get_company_summary_from_identifiers` | Yahoo Finance company summary |
| `get_next_earnings_from_identifiers` | Yahoo Finance earnings calendar |
| `get_latest_earnings_from_identifiers` | Yahoo Finance earnings history |
| `get_transcript_from_key_dev_id` | Web search for "[TICKER] earnings call transcript [quarter]" |
| `get_competitors_from_identifiers` | Yahoo Finance competitors/peers |
| `get_prices_from_identifiers` | Yahoo Finance historical prices |
| `get_financial_line_item_from_identifiers` | Yahoo Finance financials (income statement, etc.) |
| `get_capitalization_from_identifiers` | Yahoo Finance market cap |
| `get_consensus_estimates_from_identifiers` | Yahoo Finance analyst estimates |
| `get_segments_from_identifiers` | SEC EDGAR (segment disclosures from 10-Q) or Yahoo Finance |

**Step 4: Replace all Kensho `search` references in Phase 4**

Find the Phase 4 block that says "via Kensho Grounding". Replace all `search(...)` calls with `web_search(...)` or equivalent web search tool.

**Step 5: Update appendix source citation format**

Find:
```
**For raw financial data from S&P Capital IQ (revenue, EPS...):**
  - State the MCP function used... Format: `S&P Capital IQ — [function_name](...)`
```

Replace with:
```
**For raw financial data from Yahoo Finance or SEC EDGAR:**
  - State the data source and tool used. Format: `Yahoo Finance MCP — [tool_name](ticker='[TICKER]', ...)` or `SEC EDGAR MCP — [tool_name](cik='[CIK]', ...)`
```

**Step 6: Verify no remaining kfinance/Kensho references**

```bash
grep -n "kfinance\|Kensho\|Capital IQ\|S&P Global MCP" partner-built/spglobal/skills/earnings-preview-beta/SKILL.md
```

Expected: zero matches.

**Step 7: Commit**

```bash
git add partner-built/spglobal/skills/earnings-preview-beta/SKILL.md
git commit -m "feat: update earnings-preview skill to use yahoo-finance and web search"
```

---

## Task 12: Update spglobal funding-digest/SKILL.md

**Files:**
- Modify: `partner-built/spglobal/skills/funding-digest/SKILL.md`

**Step 1: Read the file**

**Step 2: Replace S&P Global data source references**

Replace any references to S&P Global tools or Capital IQ with:
- Yahoo Finance MCP for public company data
- SEC EDGAR MCP for US public company filings
- Web search for funding news, private company data, and VC/PE deal data

**Step 3: Verify**

```bash
grep -n "S&P\|Kensho\|Capital IQ\|kfinance" partner-built/spglobal/skills/funding-digest/SKILL.md
```

**Step 4: Commit**

```bash
git add partner-built/spglobal/skills/funding-digest/SKILL.md
git commit -m "feat: update funding-digest skill to use free data sources"
```

---

## Task 13: Update partner-built README files

**Files:**
- Modify: `partner-built/lseg/README.md`
- Modify: `partner-built/spglobal/README.md`

**Step 1: Update lseg/README.md**

Find the "Requirements" section:
```
- Access to the LSEG MCP Server with valid credentials
- LSEG data entitlements for the relevant product offerings
```

Replace with:
```
- Free FRED API key (register at https://fred.stlouisfed.org/docs/api/api_key.html)
- Node.js installed (for running MCP servers via npx)
```

**Step 2: Update spglobal/README.md**

Replace all subscription requirement lines. For each skill, find:
```
**Requires**: [S&P Global LLM-ready API](...) subscription
```

Replace with:
```
**Requires**: Free Yahoo Finance MCP and SEC EDGAR MCP (no subscription)
```

Also update the "How to Use" section — remove the Capital IQ Pro / LLM-ready API setup steps, replace with instructions for the free MCPs.

**Step 3: Commit**

```bash
git add partner-built/lseg/README.md partner-built/spglobal/README.md
git commit -m "docs: update partner-built READMEs to reflect free data sources"
```

---

## Task 14: Update root README.md

**Files:**
- Modify: `README.md`

**Step 1: Read the MCP integrations table (around line 108)**

**Step 2: Replace the MCP integrations table**

Find the table listing all 11 commercial MCPs and replace with:

```markdown
## MCP Integrations

| Provider | Endpoint | Cost | Capabilities |
|----------|----------|------|-------------|
| **Yahoo Finance** | stdio (local) | Free | Stock prices, fundamentals, analyst estimates, options, ETF data |
| **FRED** | stdio (local) | Free (API key) | Interest rate curves, macroeconomic indicators, inflation data |
| **SEC EDGAR** | stdio (local) | Free | US public company filings, 10-K/10-Q, segment data |

> Free API key required for FRED. Register at https://fred.stlouisfed.org/docs/api/api_key.html
```

**Step 3: Update the plugin capabilities description**

Find the line referencing all 11 data providers:
```
"Daloopa, Morningstar, S&P Global, FactSet, Moody's, MT Newswires, Aiera, LSEG, PitchBook, Chronograph, Egnyte"
```

Replace with:
```
"Yahoo Finance, FRED (Federal Reserve Economic Data), SEC EDGAR"
```

**Step 4: Commit**

```bash
git add README.md
git commit -m "docs: update README to reflect free MCP data sources"
```

---

## Task 15: Final verification

**Step 1: Check for any remaining commercial references**

```bash
grep -rn "daloopa\|morningstar\|kensho\|factset\|moodys\|mtnewswire\|aiera\|pitchbook\|chronograph\|egnyte" \
  --include="*.json" --include="*.md" . \
  --exclude-dir=".git" --exclude-dir="docs"
```

Expected: zero matches (or only in docs/plans/ as historical reference).

**Step 2: Check for remaining S&P/LSEG commercial endpoint URLs**

```bash
grep -rn "kfinance.kensho\|api.analytics.lseg\|mcp.factset\|mcp.daloopa\|mcp.morningstar\|api.moodys\|mcp.pitchbook\|chronograph.pe\|mcp-server.egnyte\|blueskyapi\|mcp-pub.aiera" \
  --include="*.json" --include="*.md" . \
  --exclude-dir=".git" --exclude-dir="docs"
```

Expected: zero matches.

**Step 3: Final commit**

```bash
git add -A
git commit -m "feat: complete replacement of commercial data sources with free alternatives

- Replaced 11 commercial MCPs with yahoo-finance, fred, and edgar
- Updated all skill files to reference free data source tools
- Rewrote partner-built LSEG and S&P Global plugins
- Updated all README and documentation files"
```

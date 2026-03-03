---
name: funding-digest
description: "Generate a polished one-page PowerPoint slide summarizing key takeaways from recent funding rounds and notable capital markets activity across a user's watched sectors or companies. Use this skill when the user asks for a deal flow summary, weekly recap, funding digest, transaction roundup, or capital markets briefing. Triggers on: 'deal flow digest', 'weekly funding recap', 'deal roundup', 'transaction summary this week', 'what happened in [sector] this week', 'capital markets update', or any request to compile recent funding activity into a briefing slide. Produces a professional single-slide PPTX with key takeaways, valuation data, and source links."
---

**AI DISCLAIMER (MANDATORY):**
You MUST include the following disclaimer text in the powerpoint footer. This is not optional — the report is incomplete without it:

> **"Analysis is AI-generated — please confirm all outputs"**

**Footer** — At the bottom of the generated slide, as a prominent yellow banner: "Analysis is AI-generated — please confirm all outputs"

---

# Weekly Deal Flow Digest

Generate an analyst-quality **single-slide PowerPoint** that summarizes key takeaways from recent funding rounds across watched sectors or companies, using web search and Yahoo Finance MCP data. Each deal links back to its original news or Crunchbase source for quick drill-down.

## When to Use

Trigger on any of these patterns:
- "Give me a deal flow digest for this week"
- "Weekly funding recap for [sector]"
- "What deals closed in [sector/companies] recently?"
- "Transaction roundup" or "deal roundup"
- "Capital markets update for my coverage universe"
- "Summarize recent funding activity"
- Any periodic briefing request about deals, raises, or rounds

## Nested Skills

This skill produces a one-slide PPTX briefing:
- **Read** `/mnt/skills/public/pptx/SKILL.md` before generating the PowerPoint (and its sub-reference `pptxgenjs.md` for creating from scratch)

## Data Sources & Search Robustness

Funding round data is sourced via **web search** (primary) and **Yahoo Finance MCP** (for public company context). Because funding data comes from news sources and databases that may have inconsistent coverage, apply these rules to avoid missing significant deals.

### Rule 0: Pre-validate companies before searching

**Before** running funding searches, confirm each company's status:
1. **Is it a public company?** Use Yahoo Finance MCP `get_company_info` — if it returns data, the company is public and you can also get financials.
2. **Is it a subsidiary?** Check recent news. Companies like DeepMind (Alphabet), GitHub (Microsoft), and BeReal (Voodoo) are subsidiaries — their capital events are tracked at the parent level. Note this context in the digest but do not search for independent funding.
3. **Is it still operating?** A quick web search for the company name can surface if it has shut down or been acquired.

**Check the `references/sector-seeds.md` file** for pre-validated seed companies, known subsidiary warnings (⚠️), and legal name variants.

### Rule 1: Never trust empty results without a fallback

If web search for "[Company] funding round [year]" returns no results for a company you expect to have data:
1. **Try the legal entity name.** Brand names can differ from legal names. Common pattern: "Together AI" → "Together Computer, Inc.", "Character.ai" → "Character Technologies, Inc.".
2. **Try alternative sources:** search Crunchbase, TechCrunch, or PitchBook coverage. Search "[Company] raises Series [X]" or "[Company] valuation [year]".
3. **Log as "no data found"** only after exhausting fallbacks — absence of search results may mean the deal wasn't yet announced or covered.

### Rule 2: Subsidiaries have no independent funding rounds

Companies that are divisions or wholly-owned subsidiaries of larger companies will not have their own independent funding rounds. Their capital events are tracked at the parent level. Note these as "acquired/subsidiary" in context.

### Rule 3: Use specific search queries, not generic ones

Broad searches return noise. Use targeted queries:
- "[Company] [Series X / seed / growth] round [year]" — find specific rounds
- "[Company] valuation [year]" — find pre/post-money data
- "[sector] startup funding [month year]" — find sector-level deal activity
- "[investor] portfolio investment [year]" — for investor-perspective queries

### Rule 4: Process companies in batches and validate

When processing large company universes (50+ companies), search in groups of 10–15. After each batch, check for companies that returned no results and run them through the fallback steps in Rule 1 before moving on.

### Rule 5: Distinguish company perspective from investor perspective

- **Company raising funds:** Search "[Company] raises [amount]" or "[Company] funding round"
- **Investor making deals:** Search "[Investor] invests in" or "[Investor] portfolio [year]"

For deal flow digests, you almost always want the company perspective.

### Rule 6: Company name variations require creative search

Search engines handle name variants well, but be creative with spelling: try "Character AI" and "Character.ai", "Runway" and "Runway ML" and "Runway AI". When uncertain, search the founder or CEO name alongside the company.

## Workflow

### Step 1: Establish Coverage & Period

Determine what the digest should cover. There are two setups:

**Returning user (has a watchlist):**
If the user has previously defined sectors or companies to track, use that list. Check conversation history for prior watchlists.

**New user:**
Ask for:

| Parameter | Default | Notes |
|-----------|---------|-------|
| **Sectors** | *(at least one)* | e.g., "AI, Fintech, Biotech" |
| **Specific companies** | Optional | Supplement sector-level coverage |
| **Time period** | Last 7 days | "This week", "last 2 weeks", "this month" |

Calculate the exact `start_date` and `end_date` from the time period.

### Step 2: Build the Company Universe

For each sector specified, build a company universe:

1. **Seed companies** from domain knowledge (see `references/sector-seeds.md`)
   - Pay attention to the ⚠️ warnings and alias notes in the seeds file — some well-known companies are subsidiaries, have been acquired, or require a specific legal name to find.

2. **Pre-validate all seeds** (Rule 0):
   - For public companies in the seed list: use Yahoo Finance MCP `get_company_info` to confirm they are operating and get sector/industry context.
   - For private companies (most VC-backed startups): do a quick web search "[Company name] [sector] startup" to confirm they are still independent and operating.
   - Triage into two buckets:
     - ✅ **Active & Independent** → proceed to universe expansion
     - ❌ **Acquired, Subsidiary, or Defunct** → note for context but exclude from funding searches

3. **Expand the universe** (using only the ✅ active seeds):
   - For public company sectors: use Yahoo Finance MCP `get_company_info` — look for listed competitors, then search web for "[sector] startup companies [year]".
   - For private/VC sectors: search web for "[sector] venture-backed companies [year]" or "[sector] top startups".

4. **Validate the expanded universe:**
   - Quick web search for any unfamiliar name to confirm operating status and sector fit.
   - Filter to match the target sector. Drop subsidiaries and defunct companies.

If the user provides specific companies, add those directly but still confirm operating status via Rule 0.

Keep the universe manageable — aim for 15–40 **active, independent** companies per sector. For a multi-sector digest, this might total 50–100+ companies.

### Step 3: Pull Funding Rounds

For all companies in the universe, use web search to find recent funding activity:

**Search strategy per company/sector:**
```
web_search("[Company] funding round [start_date] [end_date]")
web_search("[sector] startup funding rounds [month year]")
web_search("[sector] venture capital deals [month year]")
```

Process in batches of 10–15 companies if the universe is large.

**After each batch, identify companies with no results.** For any company expected to have activity:
1. Try the legal entity name or alternate search terms (see Data Sources & Search Robustness rules above).
2. Log the company as "no data found" only after exhausting fallbacks.

For sector-level discovery, also search news aggregators:
- "[sector] funding rounds this week/month"
- "[sector] deals [period]" — often surfaces rounds not found per-company

**Extract the following from each round (critical for the slide):**
- **Source URL** — link to the news article or Crunchbase page reporting the deal
- **Announcement date** — when the round was publicly announced
- **Close date** — when the round officially closed (if available)
- Amount raised
- **Pre-money valuation** (if disclosed)
- **Post-money valuation** (if disclosed)
- Lead investors
- Round type (Series A, B, C, seed, growth, etc.)
- Pricing trend (up-round / down-round / flat — compare to prior known valuation)

> **Dates are required.** The announcement and close dates must always appear in the final slide's deal table. If only one date is available, show it and mark the other as "—".

### Step 4: Pull Company Context for Notable Deals

For any company involved in a significant deal (large round, notable valuation shift), get a brief description using Yahoo Finance MCP or web search:

- For **public companies**: use Yahoo Finance MCP `get_company_info` — returns sector, industry, business description.
- For **private companies**: search web for "[Company] overview founded [year] what does it do" or look at the company's Crunchbase or press release text.

This adds context to the narrative (e.g., "The company, an AI infrastructure startup founded in 2021, is expanding into...").

### Step 5: Identify Highlights & Trends

Before designing the slide, analyze the data to surface the story:

**Flag as "Notable":**
- Rounds ≥ $100M
- Down rounds (pricing trend = down)
- New unicorns (post-money valuation crossing $1B)
- Significant valuation jumps (post-money ≥ 2x the last known valuation)
- Repeat raisers (same company raising again within 6 months)
- Unusually large investor syndicates

**Identify Trends:**
- Total capital deployed this period vs. typical (if historical data available)
- Which sub-sectors are hottest (most rounds, most capital)
- Round stage distribution (is early-stage or late-stage dominating?)
- Most active investors across the digest
- Geographic concentration
- Valuation trends (are pre-money valuations compressing or expanding?)

**Select Key Takeaways (3–5):**
Distill the most important signals into 3–5 concise bullet-style takeaways. These are the centerpiece of the slide. Each takeaway should be one sentence, punchy, and data-backed.

Examples:
- "AI sector raised $2.4B across 8 rounds — 3x the prior week, led by a $800M mega-round from [Company] at a $12B post-money valuation."
- "[Company] closed a $200M Series D at $3.5B pre-money, up from $1.8B in its Series C — signaling strong demand for AI developer tools."
- "Down-round activity ticked up: 2 of 6 late-stage rounds priced below prior valuations."

### Step 6: Generate Company Logos

For each company featured in the key takeaways or notable deals, generate a logo using a two-tier local pipeline. **Do not use Clearbit** (`logo.clearbit.com`) — it is deprecated and consistently fails. External logo CDNs (Brandfetch, logo.dev, Google Favicons) require API keys or are blocked by network restrictions. Instead, use the following approach:

#### Tier 1: `simple-icons` npm Package (3,300+ Brand SVGs, No Network Required)

The `simple-icons` package bundles high-quality SVG icons for thousands of well-known brands. It works entirely offline — no API keys, no network calls. Install it alongside `sharp` for SVG → PNG conversion:

```bash
npm install simple-icons sharp
```

**Lookup strategy:**

```javascript
const si = require('simple-icons');
const sharp = require('sharp');

// Find an icon by exact title match (case-insensitive)
function findSimpleIcon(companyName) {
    // Try exact match first
    for (const [key, val] of Object.entries(si)) {
        if (!key.startsWith('si') || !val || !val.title) continue;
        if (val.title.toLowerCase() === companyName.toLowerCase()) return val;
    }
    // Try without common suffixes (AI, Inc., Corp.)
    const stripped = companyName.replace(/\s*(AI|Inc\.?|Corp\.?|Ltd\.?)$/i, '').trim();
    if (stripped !== companyName) {
        for (const [key, val] of Object.entries(si)) {
            if (!key.startsWith('si') || !val || !val.title) continue;
            if (val.title.toLowerCase() === stripped.toLowerCase()) return val;
        }
    }
    return null;
}

// Convert SVG to PNG with the brand's official color
async function simpleIconToPng(icon, outputPath) {
    const coloredSvg = icon.svg.replace('<svg', `<svg fill="#${icon.hex}"`);
    await sharp(Buffer.from(coloredSvg))
        .resize(128, 128, { fit: 'contain', background: { r: 255, g: 255, b: 255, alpha: 0 } })
        .png()
        .toFile(outputPath);
}
```

**Coverage:** ~43% of typical deal flow companies (strong for major tech brands like Stripe, Anthropic, Databricks, Snowflake, Discord, Shopify, SpaceX, Mistral AI, Hugging Face; weaker for niche fintech, biotech, or early-stage companies).

#### Tier 2: Initial-Based Fallback via `sharp` (100% Coverage)

For companies not found in `simple-icons`, generate a clean initial-based logo as a PNG:

```javascript
async function generateInitialLogo(companyName, outputPath) {
    const initial = companyName.charAt(0).toUpperCase();
    const svg = `
    <svg width="128" height="128" xmlns="http://www.w3.org/2000/svg">
        <circle cx="64" cy="64" r="64" fill="#BDBDBD"/>
        <text x="64" y="64" font-family="Arial, Helvetica, sans-serif"
              font-size="56" font-weight="bold" fill="#FFFFFF"
              text-anchor="middle" dominant-baseline="central">${initial}</text>
    </svg>`;
    await sharp(Buffer.from(svg)).png().toFile(outputPath);
}
```

#### Complete Pipeline

```javascript
async function fetchLogo(companyName, outputDir) {
    const fileName = companyName.toLowerCase().replace(/[\s.]+/g, '-') + '.png';
    const outPath = path.join(outputDir, fileName);

    // Tier 1: Try simple-icons
    const icon = findSimpleIcon(companyName);
    if (icon) {
        await simpleIconToPng(icon, outPath);
        return { path: outPath, source: 'simple-icons' };
    }

    // Tier 2: Generate initial-based fallback
    await generateInitialLogo(companyName, outPath);
    return { path: outPath, source: 'initial-fallback' };
}
```

**Logo guidelines:**
- Save all logos to `/home/claude/logos/[company-name].png`
- All logos are 128×128 PNG with transparent backgrounds
- On the slide, display logos at 0.35"–0.5" tall — they're accents, not focal points
- Initial-fallback circles use gray (`BDBDBD`) fill with white text — consistent with the monochrome palette
- Never mix logo styles randomly — if most companies resolve to brand icons, the few fallbacks should blend in naturally

### Step 7: Generate the One-Page PPTX

Read `/mnt/skills/public/pptx/SKILL.md` and `/mnt/skills/public/pptx/pptxgenjs.md` before creating the slide.

Create a **single-slide** PowerPoint using `pptxgenjs`. The slide should be information-dense but visually clean — think "executive dashboard" not "wall of text."

#### Slide Layout

```
┌─────────────────────────────────────────────────────────────┐
│  DEAL FLOW DIGEST                                           │
│  [Period] · [Sectors]                           [Date]      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │  $X.XB  │  │  N      │  │  $X.XB  │  │  $X.XB  │       │
│  │ Raised  │  │ Rounds  │  │ Avg Pre │  │ Largest │       │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │
│                                                             │
│  KEY TAKEAWAYS                                              │
│  ─────────────────────────────────────────────────          │
│  [Logo] Takeaway 1 text goes here...                        │
│  [Logo] Takeaway 2 text goes here...                        │
│  [Logo] Takeaway 3 text goes here...                        │
│  [Logo] Takeaway 4 text goes here...                        │
│                                                             │
│  TOP DEALS                                                  │
│  ┌──────────────────────────────────────────────────────────┐│
│  │Company│Type │Announced│Closed│Amount│Pre-$│Post-$│Lead│🔗││
│  │───────│─────│─────────│──────│──────│─────│──────│────│──││
│  │ ...   │ ... │  ...    │ ...  │ ...  │ ... │ ...  │... │🔗││
│  └──────────────────────────────────────────────────────────┘│
│                                                             │
│  [Footer: Deal Flow Digest · Sources: Web Search & Yahoo Finance]│
│  [Footer: AI Disclaimer]                                    │
└─────────────────────────────────────────────────────────────┘
```

#### Design Specifications

**Color philosophy: Minimal, monochrome-first.** The slide should feel like a high-end financial brief — black, white, and gray dominate. Color is used **only** where it carries meaning (e.g., a red indicator for a down round, a green indicator for a standout metric) or where the reader would naturally expect it (company logos). Never use color for purely decorative purposes like background fills, accent bars, or gradient effects.

**Color palette — Monochrome Executive:**
- Primary background: `FFFFFF` (white) — clean, open slide background
- Header bar: `1A1A1A` (near-black) — strong contrast for the title region
- Primary text: `1A1A1A` (near-black) — all body text, stat numbers, takeaways
- Secondary text: `6B6B6B` (medium gray) — labels, captions, footer, date stamps
- Borders & dividers: `D0D0D0` (light gray) — subtle structural lines, card outlines, table borders
- Card backgrounds: `F5F5F5` (off-white / very light gray) — stat card fills, alternating table rows
- Link text: `2B5797` (muted blue) — source deal links in the table (the only blue on the slide)
- **Semantic color (sparingly):**
  - Down rounds or negative signals: `C0392B` (muted red) — use only as a small dot, tag, or single-word highlight, never as a fill or background
  - Standout positive metrics (new unicorn, outsized round): `2E7D32` (muted green) — same minimal usage: a dot, a small tag, or a single highlighted number
  - If no data points warrant a color indicator, **use no color at all**. A fully monochrome slide is perfectly correct.

**Typography:**
- Title: 28–32pt, bold, white on near-black header bar
- Stat numbers: 36–44pt, bold, near-black
- Stat labels: 10–12pt, medium gray (`6B6B6B`)
- Takeaway text: 12–14pt, near-black, left-aligned
- Table text: 9–11pt, near-black with gray (`6B6B6B`) for secondary columns
- Link text: 9–10pt, muted blue (`2B5797`)
- Footer: 8pt, medium gray

**Stat Cards (top row):**
- 4 key metrics as large-number callouts: Total Raised, # Rounds, Avg Pre-Money Valuation, Largest Round
- Each in a card with `F5F5F5` fill and a thin `D0D0D0` border — no shadow, no color fills
- If a stat is surprising or extreme (e.g., 3x normal volume, a record deal), a small colored dot or underline may be placed next to that single number — otherwise keep fully monochrome
- If pre-money valuations are mostly undisclosed, substitute with a different metric (e.g., Median Round Size, # New Unicorns)

**Key Takeaways (middle section):**
- 3–5 one-line takeaways, each prefixed with the relevant company logo (small, ~0.35" tall)
- If no logo available, use a **gray circle** with the company initial in white — not a colored circle
- Left-aligned, with enough spacing to breathe
- Down-round or negative takeaways may use a small red dot prefix; otherwise no color
- Include valuation context where available (e.g., "at a $5B post-money valuation")

**Top Deals Table (bottom section):**
- Compact table showing the 4–6 most notable deals
- Columns: Company, Type (Series X), Announced (date), Closed (date), Amount ($M), Pre-Money ($M), Post-Money ($M), Lead Investor, Deal Link
- **Announced** and **Closed** columns show dates in `MMM DD` format (e.g., "Jan 15"). These columns are required and must always be present. If a date is not available, show "—".
- The **Deal Link** column contains a clickable "View →" text linking to the original news article, Crunchbase profile, or press release URL found during web search in Step 3. Use the most authoritative source available (company press release > major tech news outlet > Crunchbase).
- If pre-money or post-money valuation is not disclosed, show "—" in that cell
- Header row with near-black (`1A1A1A`) fill and white text; alternating rows in `F5F5F5` and `FFFFFF`
- **Center the table horizontally** on the slide. Calculate the table's total width, then set `x` so it is centered within the slide width: `x = (slideWidth - tableWidth) / 2`. For a 16:9 layout (13.33" wide), if the table is 12" wide, use `x = 0.67`. Never left-align the table to the slide edge.
- Keep it tight — this is a reference, not the focal point
- No colored fills in table cells. If a deal is a down round, a small red text tag "(↓ down)" may appear next to the amount — that is the only permitted color in the table.

**Deal Link Implementation (pptxgenjs):**
In pptxgenjs, hyperlinks are added to table cells using the `options.hyperlink` property on the cell object:
```javascript
// Table cell with deal source link (news article, Crunchbase, or press release URL from web search)
{
  text: "View →",
  options: {
    hyperlink: {
      url: sourceUrl  // URL retrieved via web search in Step 3
    },
    color: "2B5797",
    fontSize: 9,
    fontFace: "Arial"
  }
}
```

**Table Centering (pptxgenjs):**
Always center the deal table on the slide. Calculate the x position dynamically:
```javascript
const SLIDE_W = 13.33; // 16:9 slide width
const TABLE_W = 12.5;  // total table width (sum of all column widths)
const TABLE_X = (SLIDE_W - TABLE_W) / 2; // ≈ 0.42"

slide.addTable(tableRows, {
  x: TABLE_X,
  y: tableY,
  w: TABLE_W,
  colW: [1.8, 0.9, 0.9, 0.9, 1.0, 1.1, 1.2, 1.6, 0.7], // Company, Type, Announced, Closed, Amount, Pre-$, Post-$, Lead, Link
  // ... other options
});
```
Adjust `colW` values as needed, but always recompute `TABLE_X` from `(SLIDE_W - sum(colW)) / 2` to keep the table centered.

**Footer:**
- Small text in medium gray: "Deal Flow Digest · [Period] · Sources: Web Search & Yahoo Finance MCP · Generated [Date]"

**General color rules (enforce strictly):**
- Company logos are the only "full color" elements on the slide — they appear as-is from the source.
- Deal links use muted blue (`2B5797`) — this is the only non-monochrome text color besides semantic red/green.
- Outside of logos and links, the slide should look correct printed on a black-and-white printer.
- Never apply color to backgrounds, accent bars, decorative shapes, or section dividers.
- When in doubt, leave it gray.

#### Code Structure

```javascript
const pptxgen = require("pptxgenjs");
const pres = new pptxgen();
pres.layout = "LAYOUT_16x9";
pres.title = "Deal Flow Digest";

const slide = pres.addSlide();
const SLIDE_W = 13.33; // 16:9 slide width in inches

// 1. Dark header bar with title and period
// 2. Stat cards row (4 cards: Total Raised, # Rounds, Avg Pre-Money, Largest Round)
// 3. Key takeaways section with logos (include valuation context)
// 4. Top deals table with Announced, Closed, Pre-Money, Post-Money columns and source deal links
//    - Center the table: x = (SLIDE_W - tableWidth) / 2
// 5. Footer

pres.writeFile({ fileName: "/home/claude/deal-flow-digest.pptx" });
```

Use factory functions (not shared objects) for shadows and repeated styles per the pptxgenjs pitfalls guidance.

### Step 8: QA the Slide

Follow the QA process from the PPTX skill:

1. **Content QA:** `python -m markitdown deal-flow-digest.pptx` — verify all text, numbers, company names, valuation figures, and deal links are correct
2. **Visual QA:** Convert to image and inspect:
   ```bash
   python /mnt/skills/public/pptx/scripts/office/soffice.py --headless --convert-to pdf deal-flow-digest.pptx
   pdftoppm -jpeg -r 200 deal-flow-digest.pdf slide
   ```
   Check for overlapping elements, text overflow, alignment issues, low-contrast text, logo sizing problems, and that deal link text is visible.
3. **Link QA:** Verify that the source URLs in the table are correctly formatted and point to the actual news articles, Crunchbase pages, or press releases found during web search.
4. **Fix and re-verify** — at least one fix-and-verify cycle before declaring done.

### Step 9: Present Results

1. Copy the final `.pptx` to `/mnt/user-data/outputs/`
2. Use `present_files` to share the slide
3. Provide a 2–3 sentence verbal summary:
   - "Your digest covers X rounds totaling $Y raised across [sectors]."
   - Call out the single most notable deal and its valuation
   - Flag any concerning trends (down rounds, valuation compression, etc.)

## Error Handling

### Search & Data Failures
- **No results for a known company:** Try the legal entity name and alternative search terms. Common brand→legal mismatches: Together AI → "Together Computer, Inc.", Character.ai → "Character Technologies, Inc.", Runway ML → "Runway AI, Inc.". Also try searching founder names or lead investor names alongside the company.
- **Subsidiary companies:** DeepMind, GitHub, Instagram, WhatsApp, YouTube, BeReal, etc. are subsidiaries — they have zero independent funding rounds. Note these as "acquired/subsidiary" in context but do not report them as "no activity."
- **Defunct companies:** Companies like Convoy (shut down Oct 2023) will have no new funding activity. The `references/sector-seeds.md` file flags these — check it before including a company.
- **Paywalled sources:** If a search result leads to a paywalled article (WSJ, FT, Bloomberg), look for the same deal announcement on the company's press release page, Crunchbase, or in TechCrunch/Reuters coverage.
- **Wrong search perspective:** If company-focused searches return no results, try searching from the investor perspective: "[Investor firm] new investment [sector] [year]".

### Data Quality Issues
- **No activity in period:** If a sector had zero funding rounds, note this explicitly on the slide ("No transactions recorded in [Sector] during the period") — absence of activity is itself informative.
- **Sparse valuation data:** If pre-money and post-money valuations are undisclosed for most transactions, note the data limitation in a footer annotation and use "—" in the table. Adjust the stat card to show a different metric (e.g., Median Round Size) instead of Avg Pre-Money.
- **Logo retrieval failures:** The `simple-icons` npm package provides ~43% coverage for typical deal flow companies. For the remainder, use the `sharp`-generated initial-based fallback. Keep a consistent icon style — don't mix random approaches. If `simple-icons` or `sharp` fail to install, fall back to pptxgenjs shape-based initials (gray ellipse + white text overlay) which require no external dependencies.
- **Too many deals for one slide:** If there are more than 6 notable deals, show the top 6 in the table and add a footnote: "+N additional deals not shown." Prioritize by deal size.
- **Large universes:** For multi-sector digests with 100+ companies, batch all API calls in groups of 15–20. Prioritize depth on notable deals over completeness on minor ones.
- **Stale seeds:** If competitor expansion returns very few results for a sector, the seed companies may be too niche. Broaden by adding 2–3 more well-known names and re-expanding.
- **Broken or inaccessible source URLs:** If a news article URL found via web search returns a 404 or paywall, find an alternative source for the same deal and update the link. Do not include a broken link in the table — omit the link cell and note "Source unavailable" if no alternative can be found.

## Example Prompts

- "Give me a weekly deal flow digest for AI and fintech"
- "Summarize this week's funding in biotech"
- "Deal roundup for my coverage — cybersecurity, cloud infrastructure, and dev tools — last 2 weeks"
- "What happened in venture this week across all sectors I follow?"
- "Quick deal flow slide for climate tech this month"
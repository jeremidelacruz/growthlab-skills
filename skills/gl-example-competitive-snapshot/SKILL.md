---
name: gl-example-competitive-snapshot
description: >
  Generic competitive snapshot for paid media. Triggers on: "competitive snapshot",
  "quick competitor overview", "competitive brief", "analyze my competitors",
  "what are competitors doing in ads". Produces a structured markdown brief with
  positioning, ad themes, and tactical observations — no proprietary process,
  no client data. Use as a starting point or a demo of the gl- skill format.
version: 0.1.0
author: GrowthLab (growthlab.mx)
license: MIT
---

# gl-example-competitive-snapshot

> **Purpose:** Rapid competitive snapshot for any brand or market. Generic and shareable — no GrowthLab proprietary playbook included. Shows the gl- skill format: frontmatter → context → inputs → process → output.

---

## Before starting

If you have a `.agents/product-marketing.md` or `CLAUDE.md` in this project, read it now to understand the client's industry, ICP, and positioning. Use that context to filter which competitors are relevant.

---

## Inputs

Provide at least one of:

- **Brand name or URL** of the brand you're analyzing competitors FOR.
- **Competitor list** (comma-separated names or URLs) — optional; Claude will suggest candidates if not provided.
- **Market / geography** — e.g., "CDMX B2C insurance", "US SaaS HR tools".
- **Ad platforms** to cover — e.g., Meta, Google Search, both.

---

## Process

### Step 1 — Identify competitors

If no competitor list is provided:
- Use your knowledge of the market to propose 3–5 direct competitors.
- Ask the user to confirm or adjust the list before proceeding.

### Step 2 — Gather signals

For each competitor, collect available signals:

- **Meta Ads Library** (if `mcp__meta-ads__ads_library_search` is available): search active ads for each competitor. Note: ad count, primary creative themes, CTAs, offers.
- **Google Ads / Search** (if `mcp__google-ads__search` is available): look for branded terms, key search terms they appear on.
- **Organic / web signals**: homepage positioning, tagline, primary value proposition.

If MCP tools are not available, note the gap and proceed with what is available (manual URLs, user-provided screenshots, public ad library URLs).

### Step 3 — Analyze

For each competitor, answer:

1. **Positioning:** What core promise do they lead with?
2. **Audience:** Who are they targeting (inferred from creative/copy)?
3. **Ad themes:** What 2–3 themes or hooks appear most in their ads?
4. **Offer/CTA:** What conversion action are they pushing (lead gen, trial, purchase, call)?
5. **Tone:** Rational/emotional? Professional/casual? Fear-based/aspiration-based?

### Step 4 — Synthesize

Identify:

- **Market white space:** positioning angles none of the competitors own.
- **Dominant themes:** what everyone is saying (so you can differentiate or match).
- **Tactical observations:** budget signals, creative format preferences, landing page patterns.

---

## Output

Produce a structured markdown document with:

```
# Competitive Snapshot — [Market / Brand]
Date: YYYY-MM-DD

## Competitors Analyzed
[List]

## Per-Competitor Summary
[One section per competitor: positioning, audience, themes, offer, tone]

## Market-Level Observations
### What everyone is saying
### White space / differentiation angles
### Tactical signals

## Recommended next steps
[3 bullet points max — specific, actionable]
```

Output language should match the user's preference. If a `product-marketing.md` is present and specifies Spanish (MX), output in Spanish.

---

## References

- Based on the open-source `competitor-profiling` and `competitors` skill patterns from the Claude Code community.
- For a production version with GrowthLab's proprietary scoring, PPC Reveal integration, and deck output, see `gl-paid-competitive-research` in the private `growthlab-os` repo.

---

## Notes for skill developers

This file demonstrates the gl- skill format:

- **Frontmatter:** `name`, `description` (with English triggers), `version`, `author`, `license`.
- **Before starting:** read context files.
- **Inputs:** explicit, enumerated.
- **Process:** numbered steps with conditional branching (MCP available / not available).
- **Output:** concrete template — not just "provide analysis".
- **References:** cite the base skill(s) you forked or merged from.

# Skills — Curation Guide & Attribution

This directory contains:
1. **GrowthLab-authored skills** (`gl-*`) — original, shareable, no proprietary process or client data.
2. **References to curated third-party skills** — we evaluated these and recommend them; install from the original source.

---

## Curation Approach

We evaluated 180+ community skills across the Claude Code ecosystem. Our criteria:

- **Battle-tested in real paid-media work** — not just demos.
- **Specific output** — produces a deliverable (deck, report, brief, copy), not just commentary.
- **Compatible with agency workflows** — works alongside MCP tools (Meta Ads, Google Ads, etc.).
- **Safe to run autonomously** — minimal hallucination risk on structured tasks.

We do NOT re-host third-party skills verbatim. We link to the original source so authors get credit and users get the most up-to-date version.

---

## GrowthLab-Authored Skills (in this repo)

| Skill | Description | File |
|---|---|---|
| `gl-example-competitive-snapshot` | Generic competitive snapshot — demo of the gl- skill format | [gl-example-competitive-snapshot/SKILL.md](gl-example-competitive-snapshot/SKILL.md) |

---

## Curated Third-Party Skills

> Install from the original source to get the latest version and respect the author's license.

### Paid Media & Research

| Skill | Author | What it does | Install |
|---|---|---|---|
| `competitor-profiling` | Anthropic / community | Structured competitor research with positioning analysis | Built-in Claude Code skill |
| `competitors` | Anthropic / community | Rapid competitor overview from a URL or name | Built-in Claude Code skill |
| `ad-creative` | Anthropic / community | Ad copy generation for paid social and search | Built-in Claude Code skill |
| `ads` | Anthropic / community | Campaign briefing and ad strategy | Built-in Claude Code skill |

### SEO & Content

| Skill | Author | What it does | Install |
|---|---|---|---|
| `seo-audit` | Anthropic / community | Technical + content SEO audit with prioritized fixes | Built-in Claude Code skill |
| `ai-seo` | Anthropic / community | AI Engine Optimization (AEO/GEO) — visibility in AI citation engines | Built-in Claude Code skill |
| `schema` | Anthropic / community | Structured data / JSON-LD schema implementation | Built-in Claude Code skill |
| `content-strategy` | Anthropic / community | Editorial calendar and content pillar planning | Built-in Claude Code skill |
| `copywriting` | Anthropic / community | Persuasive copy with AIDA / PAS / Cialdini frameworks | Built-in Claude Code skill |

### Analytics & CRO

| Skill | Author | What it does | Install |
|---|---|---|---|
| `analytics` | Anthropic / community | GA4 / reporting analysis and insights | Built-in Claude Code skill |
| `cro` | Anthropic / community | Conversion rate optimization audit and recommendations | Built-in Claude Code skill |
| `benchmark` | Anthropic / community | Industry benchmark comparison for paid media KPIs | Built-in Claude Code skill |

### Strategy & Planning

| Skill | Author | What it does | Install |
|---|---|---|---|
| `marketing-plan` | Anthropic / community | Full marketing plan from brief to calendar | Built-in Claude Code skill |
| `customer-research` | Anthropic / community | ICP / persona development from interviews or data | Built-in Claude Code skill |

---

## How "Built-in Claude Code skill" works

Many skills above ship with Claude Code or the community skill registry. To use them:

```
# In Claude Code chat:
/seo-audit
/competitor-profiling
```

If a skill isn't pre-installed, check the Claude Code community registry or ask Claude Code to install it via `/skills install <name>`.

---

## Attribution

All third-party skills are the intellectual property of their respective authors. GrowthLab lists them here as references only. If you are an author and want your skill removed from this list or have attribution corrections, open an issue or email hola@growthlab.mx.

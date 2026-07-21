# Skills

GrowthLab-original skills — each one self-contained, MIT-licensed, and safe to drop into any Claude Code project. No proprietary process, no client data, no dependency on other skills.

---

## Available skills

| Skill | Description | File |
|---|---|---|
| `product-marketing-md` | Turn scattered knowledge about your product and your customer into a `product-marketing.md` context file any LLM reads before it writes ads, copy, emails, or posts. Interviews you (or auto-drafts from your site/repo), captures verbatim customer language, and enforces one rule: no invented numbers. Pairs with `design-md`. | [product-marketing-md/SKILL.md](product-marketing-md/SKILL.md) |
| `design-md` | Convert a brand identity source — PDF manuals + live sites + W3C tokens — into a `design.md` rules file any LLM can read and verify itself against, plus an optional DTCG `design-tokens.json`. Extracts exact hex values and real embedded font names even from vectorized PDFs. | [design-md/SKILL.md](design-md/SKILL.md) |

---

## Install

Copy a skill's folder into your project's `.claude/skills/<skill-name>/`, restart Claude Code, and invoke it with `/<skill-name>` or by describing the task.

---

## On attribution

Every skill here is written from scratch by GrowthLab — we do not re-host third-party skills. Where a skill invokes an external **tool** (e.g. PyMuPDF, Poppler, fontTools), that tool is credited with its license in the skill's own "References & credits" section. Where we **adapt an idea or format** from another skill (e.g. `design-md` adapts its live-site capture and W3C DTCG token output from [`anydesign`](https://github.com/uxKero/anydesign) by @uxKero, MIT — reimplemented in our own code, not copied), that source is credited the same way. If you're an author and believe something here needs correction, open an issue or email hola@growthlab.mx.

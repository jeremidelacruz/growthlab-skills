# GrowthLab Skills

> Skills originales de GrowthLab (Monterrey) — battle-tested en trabajo real de paid media.
> Escritos desde cero, sin dependencias, MIT. Tuyos para usar.

---

## What is this?

GrowthLab is a performance-marketing agency based in Monterrey, México. We run paid media (Google Ads, Meta) for B2C and B2B clients across Mexico, and we build our delivery on top of Claude Code.

This repository is our **public skill library** — original skills we wrote for our own workflow and cleaned up to share. They're self-contained (no dependency on other skills), MIT-licensed, and useful to anyone, not just marketers.

We studied the ecosystem — hundreds of community skills and agents — and kept what worked as *inspiration*. But everything published here is **GrowthLab-original code**: we don't re-host anyone else's skill. The external tools each skill invokes (PyMuPDF, Poppler, etc.) are credited with their licenses inside each skill.

---

## Skills

| Skill | What it does |
|---|---|
| [`design-md`](skills/design-md/SKILL.md) | Converts a brand identity source — PDF manuals + live sites (multi-viewport Playwright capture) + images — into a `design.md` rules file any LLM can read and verify itself against, plus an optional W3C DTCG `design-tokens.json`. Extracts exact hex values and the real embedded font names even from fully-vectorized PDFs. Self-contained, no dependencies. |

More skills coming — this repo grows as we clean up and release more of our internal toolkit.

---

## How to install a skill in Claude Code

1. Open the skill folder in [`skills/`](skills/) and copy its whole directory (the `SKILL.md` + `references/`) into your project's `.claude/skills/<skill-name>/`.
2. Restart Claude Code (or reload skills) so it picks up the new skill.
3. Invoke it by typing `/<skill-name>` in chat, or just describe the task — the skill's trigger phrases will match.

> **Tip:** `design-md` pairs well with a `CLAUDE.md` that tells your agent *"Before any creative task, read `design.md` and follow it exactly."* That one line is what turns "make it on-brand" into a rule the model can actually check itself against.

---

## Why we build this way

Most teams use AI for marketing by pasting a prompt into a chatbot and hoping. We don't. The difference is **context**: our agents read a `design.md` (visual identity) and a product-marketing context file *before* they make anything. These skills are the tools that produce and enforce that context.

We're an AI-forward agency — Claude Code is part of our real delivery workflow, not a demo.

---

## Links

- Website: [growthlab.mx](https://growthlab.mx)
- Contact: hola@growthlab.mx

---

## License

MIT — see [`LICENSE`](LICENSE). Everything here is GrowthLab-original content. The third-party *tools* a skill invokes (e.g. PyMuPDF, Poppler, fontTools) keep their own licenses, noted inside each skill's credits.

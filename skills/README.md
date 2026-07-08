# Skills

GrowthLab-original skills — each one self-contained, MIT-licensed, and safe to drop into any Claude Code project. No proprietary process, no client data, no dependency on other skills.

---

## Available skills

| Skill | Description | File |
|---|---|---|
| `design-md` | Convert a brand identity manual (PDF, website, images) into a `design.md` rules file that any LLM can read — and verify itself against. Extracts exact hex values and real embedded font names even from vectorized PDFs. | [design-md/SKILL.md](design-md/SKILL.md) |

---

## Install

Copy a skill's folder into your project's `.claude/skills/<skill-name>/`, restart Claude Code, and invoke it with `/<skill-name>` or by describing the task.

---

## On attribution

Every skill here is written from scratch by GrowthLab — we do not re-host third-party skills. Where a skill invokes an external **tool** (e.g. PyMuPDF, Poppler, fontTools), that tool is credited with its license in the skill's own "References & credits" section. If you're an author and believe something here needs correction, open an issue or email hola@growthlab.mx.

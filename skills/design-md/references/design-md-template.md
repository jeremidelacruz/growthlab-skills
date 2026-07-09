# `design.md` template

Two parts: a blank template to copy, then a filled neutral example.

> GrowthLab-original content. MIT licensed (see repo `LICENSE`).

> `design.md` is the **primary, human-readable** deliverable. For a machine-readable companion (Style Dictionary / Figma Variables / Tokens Studio), also emit `design-tokens.json` — see skill Step 5b and `design-tokens-template.json`. Keep the two in sync; if they disagree, this file wins.

Rules for filling it in:

- Mark unverified values `UNVERIFIED`, eyedropped values `APPROX`, and anything you invented `PROPOSED`.
- Every hex that appears anywhere in the file — including in Do/Don't and the checklist — must be declared in `## Palette`. **Forbidden colors are declared there too**, with "Forbidden" as their Role. Otherwise the orphan-color check (cookbook §8) will flag them, correctly.
- Delete rows you cannot fill. An empty row is worse than a missing one: it looks like a rule and enforces nothing.

---

## Part 1 — Blank template

````markdown
# design.md — <BRAND NAME>

Source: <brand-manual.pdf, pages 4–17> · Extracted: <YYYY-MM-DD>
Conflicts: <e.g. "Manual says #XXXXXX, site CSS says #YYYYYY. Manual wins for print, site for digital.">

Read this file before any creative task. Follow it exactly. Verify against the checklist before shipping.

## Palette

| Token | Hex | Role |
|---|---|---|
| `<token>` | `#RRGGBB` | <Primary / Secondary / Accent / Surface / Border — and what it is used for> |
| `<token>` | `#RRGGBB` | <role> |

**Usage rules**
- Must appear on every deliverable: `<token>`
- Accent only, never a background: `<token>`, capped at <N>% of surface area
- Tints/shades defined by the system: `<token-60>` = `#RRGGBB` (<60% of `<token>` over `<surface>`)

**Forbidden**

| Token | Hex | Role |
|---|---|---|
| `pure-black` | `#000000` | Forbidden. Use `<ink token>` instead. |
| `<near-miss>` | `#RRGGBB` | Forbidden. A near-miss of `<token>`; never substitute. |

**Print references** (verbatim from the manual — not converted)
- `<token>`: CMYK <c/m/y/k> · Pantone <PMS xxx>

## Typography

| Role | Real typeface | Licence | Renders as | Stack |
|---|---|---|---|---|
| Primary | <Name> | <Commercial / OFL / Apache> (<installed / not installed>) | <what actually renders> | `"<Real>", "<Fallback>", <generic>` |
| Secondary | <Name> | <...> | <...> | `"<Real>", "<Fallback>", <generic>` |

<If a real typeface is not installed, say so here in one sentence, plainly.>

**Type scale**

| Level | Size | Weight | Usage |
|---|---|---|---|
| Display | <48px> | <700> | <Hero only. One per layout.> |
| H1 | <32px> | <700> | <Page title.> |
| H2 | <24px> | <600> | <Section headings.> |
| Body | <16px> | <400> | <Default. Line-height <1.5>.> |
| Caption | <13px> | <400> | <Labels, captions, table headers.> |

## Logo & Isotype

**Canonical files** — use these, never a reproduction.
- Full lockup: `<assets/logo-full.svg>`
- Isotype only: `<assets/isotype.svg>`
- Reversed / knockout: `<assets/logo-reversed.svg>`

**Construction**
- Clear space: <1× the isotype height on all sides>
- Minimum size: <24px screen / 10mm print>
- Approved lockups: <horizontal (default), stacked (narrow layouts), isotype-only (≥ 3rd mention)>
- Approved color versions: <full color on light, reversed on `<token>`, mono black, mono white>

**Never**
- Never redraw, trace, or rebuild the mark. Place the supplied vector.
- Never rotate, skew, or stretch.
- Never recolor outside the approved versions.
- Never place on a busy background or a photograph without an approved scrim.
- Never add effects — shadow, glow, outline, gradient.
- Never alter the spacing between isotype and wordmark.
- <transcribe any further prohibitions from the manual, verbatim>

## Voice

**Tagline:** <verbatim, with exact punctuation and capitalization>

**Tone:** <3–6 adjectives; keep any contrasts the manual states, e.g. "confident, not arrogant">

**Writing rules**
- <Sentence case in headings. No title case.>
- <Oxford comma: yes/no.>
- <First person plural ("we"), never singular.>
- <Contractions: allowed / not allowed.>
- Signature spellings: <ProductName (never PRODUCTNAME), ...>

**Banned words:** <words the brand never says>

## Do / Don't

| Do | Don't |
|---|---|
| <Use `#RRGGBB` for all body text> | <Use pure black `#000000`> |
| <Set headings in the Primary stack> | <Substitute a "similar" typeface> |
| <Place the supplied `assets/logo-full.svg`> | <Redraw, trace, or recolor the mark> |
| <Keep accent coverage under N%> | <Use more than two accent colors per layout> |

## Deliverable checklist

- [ ] Every color used appears in the Palette table
- [ ] `<required token>` appears on the deliverable
- [ ] Accent coverage ≤ <N>% of surface area
- [ ] All type is set in the Primary or Secondary stack
- [ ] Logo is the supplied `<assets/logo-full.svg>` — not redrawn
- [ ] Clear space around the logo ≥ <1× isotype height>
- [ ] No banned word appears in the copy
- [ ] <one more falsifiable, brand-specific item>
````

---

## Part 2 — Worked example

**This is GrowthLab's own `design.md`, produced with this skill.** The values are real and public
(they appear on growthlab.mx and in the brand's own artwork) — a brand's visual identity is meant
to be seen. It is shown here as a genuine, non-fictional example of the output. Do not reuse
GrowthLab's identity for your own brand; replace every value with your own.

````markdown
# design.md — GrowthLab

Source: growthlab brand manual (21-page guide) · Extracted: 2026-06-19
Fonts confirmed by reading the manual's embedded font table (the manual is fully vectorized;
`get_text()` returns nothing — see the skill's Step 2). Primary/secondary typefaces are
commercial and NOT installed on the render machine; everything below renders in the fallbacks.

Read this file before any creative task. Follow it exactly. Verify against the checklist before shipping.

## Palette

| Token | Hex | Role |
|---|---|---|
| `tech-blue` | `#1161EE` | Primary. Headings, key data, CTAs, brand anchor. Must appear on every deliverable. |
| `ink` | `#111111` | Body text. |
| `white` | `#FFFFFF` | Default background. |
| `cloud` | `#F4F4F4` | Surface / card background. |
| `slate` | `#6B6B6B` | Supporting text, captions. |
| `awareness-red` | `#F84E37` | Accent only. Alerts, urgency, competitor callouts. Never a background on multi-page docs. |
| `satisfaction-yellow` | `#CBF834` | Accent only. Highlights, positive metrics, wins. |
| `growth-purple` | `#AF24F6` | Accent only. Premium / AI-tech / innovation sections. |
| `yellow-contrast` | `#9BC400` | Computed darker yellow-green. Use instead of `satisfaction-yellow` when yellow needs contrast on a white/light background (icons, thin strokes). |

**Usage rules**
- Must appear on every deliverable: `tech-blue` — at minimum in headings or key data.
- White-on-`tech-blue` is the primary CTA combination.
- Accents (`awareness-red`, `satisfaction-yellow`, `growth-purple`) for emphasis only — never a full background on a multi-page document.
- `satisfaction-yellow` on white fails contrast — swap to `yellow-contrast` for any small or thin element.

**Forbidden**

| Token | Hex | Role |
|---|---|---|
| `mustard` | `#E1B12C` | Forbidden. An old off-brand deck used a mustard/warm-beige palette; never reintroduce it. |

**Notes**
- Fonts are stated below; there are no Pantone/CMYK values transcribed here because this file governs digital deliverables. Add them from the manual if you produce print.

## Typography

| Role | Real typeface | Licence | Renders as | Stack |
|---|---|---|---|---|
| Primary | Agenda (Font Bureau) | Commercial (not installed) | Hanken Grotesk | `"Agenda", "Hanken Grotesk", system-ui, sans-serif` |
| Secondary | Gotham (Hoefler&Co) | Commercial (not installed) | Montserrat | `"Gotham", "Montserrat", system-ui, sans-serif` |

Neither Agenda nor Gotham is installed on the machine producing these deliverables, so everything
rendered here is actually set in **Hanken Grotesk** and **Montserrat** (both OFL). These are
stylistic substitutes, not metric-compatible — line breaks differ from artwork set in the licensed
fonts. Anything final for print must be reset by someone holding the Agenda/Gotham licences. Drop
the licensed `.woff2` files in and the stack picks them up automatically (real font is first).

**Type scale**

| Level | Size | Weight | Usage |
|---|---|---|---|
| Display | 40–56px | 700 | Deck / cover titles. One per layout. |
| H1 | 28–32px | 700 | Slide or section title. |
| H2 | 20–24px | 600 | Subsection heading. |
| H3 | 16–18px | 600 | Card / table heading. |
| Body | 14–16px | 400 | Default body copy. |
| Caption | 12px | 400 | Labels, footnotes, sources. |

## Logo & Isotype

**Canonical files** — use these, never a reproduction.
- Full lockup: `assets/growthlab-logo.svg`
- Isotype only: `assets/growthlab-isotype.svg`
- Reversed / knockout (white on blue/dark): `assets/growthlab-logo-white.svg`

**Construction**
- The isotype is a solid `tech-blue` square containing TWO white arrows in an L — one pointing UP, one pointing RIGHT, stems joined. This "positive XY displacement" (growth on both axes) is the mark.
- Clear space: 1× the isotype width on all sides; 3× for large placements.
- Minimum size: 24px height on screen; 6mm in print.
- Wordmark "GrowthLab" set in Agenda Bold (renders Hanken Grotesk 800), tight tracking, ink — isotype left, wordmark right.

**Never**
- Never redraw, trace, or rebuild the mark. Place the supplied vector.
- **Never render the isotype as a single diagonal ↗ arrow.** It is always the two-arrow (up + right) composition.
- **Never show an arrow pointing down, left, or backwards** — not even in a data chart implying a decline. Show flat or an alternative indicator instead.
- Never rotate, skew, stretch, or add effects (shadow, glow, gradient).
- Never place the mark on a busy background without adequate contrast.

## Voice

**Anchor:** Ingresos y ganancias — no métricas de vanidad.

**Tone:** direct, scientific, ROI-focused. Anti-fluff. Every claim backed by data. Keywords: neat, precise, sharp, minimalist, contemporary, methodic, analytical.

**Writing rules**
- Client-facing copy is Spanish (Mexico) — es-MX, not es-ES.
- Lead with revenue / CPA / ROAS impact — never with impressions or reach.
- Use precise numbers; never vague phrases like "mejoró significativamente".
- Every recommendation includes its rationale (why it moves revenue).

**Banned words:** revolucionario, increíble, disruptivo, "el futuro es ahora", and any vanity metric (reach, impressions, likes) framed as a *result* rather than context.

## Do / Don't

| Do | Don't |
|---|---|
| Use `#1161EE` on every deliverable (headings / key data) | Ship a page with no `tech-blue` on it |
| Pair white text on `#1161EE` for CTAs | Use an accent as a full-page background |
| Use `#9BC400` for small yellow elements on white | Use `#CBF834` for thin strokes on white (fails contrast) |
| Set type in the Agenda/Gotham stack (renders Hanken/Montserrat) | Substitute a "similar" geometric sans |
| Place `assets/growthlab-logo.svg` | Redraw the isotype or use a single diagonal arrow |
| Draw growth arrows up-and-right | Show any arrow pointing down — even for a real decline |
| Lead the report with CPA / ROAS | Lead with reach or impressions |

## Deliverable checklist

- [ ] Every color used appears in the Palette table
- [ ] `tech-blue` (`#1161EE`) appears on the deliverable
- [ ] No accent color is used as a full-page background
- [ ] Small yellow elements on white use `#9BC400`, not `#CBF834`
- [ ] All type is set in the Agenda or Gotham stack
- [ ] Logo is the supplied two-arrow isotype — not redrawn, not a single diagonal arrow
- [ ] No arrow points down, left, or backwards
- [ ] es-MX copy leads with a revenue/CPA/ROAS metric, not a vanity metric
````

---

## Notes on the example

- **This is a real, published identity — used with permission because it is GrowthLab's own.** Your `design.md` should be built the same way from *your* manual. Never copy another brand's values.
- **The Typography row is the honest part, and it is real here.** Agenda and Gotham are genuinely commercial and genuinely not installed on the render machine, so the output genuinely renders in Hanken Grotesk and Montserrat. Writing "Primary: Agenda" and stopping would make every downstream artifact silently wrong. The stack lists the real font first so it activates automatically the day the licensed files are installed.
- **`yellow-contrast` (`#9BC400`) is computed, not eyeballed** — a darker yellow-green derived so the brand's bright `satisfaction-yellow` stays legible on white. Derived values get computed and marked; transcribed values get transcribed.
- **The "never point down" logo rule looks odd until you see the isotype** — the mark *is* two growth arrows, so a downward arrow anywhere in a deliverable contradicts the identity. That is exactly the kind of brand-specific, checkable rule a `design.md` exists to carry.
- **Every Do/Don't row is checkable by looking at the output.** None say "be bold" or "feel premium."

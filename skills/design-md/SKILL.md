---
name: design-md
description: >
  Convert a brand identity manual (PDF, images, or a live website) into a `design.md`
  file that any LLM can read and respect. Triggers on: "create design.md",
  "brand manual to markdown", "make my brand guidelines AI-readable",
  "convert identity manual", "extract brand colors and fonts", "brand tokens from PDF",
  "turn our style guide into rules for Claude". Extracts exact hex values, real embedded
  font names (even from vectorized PDFs), logo rules, and voice — then emits a
  self-verifiable rules file. Self-contained: no other skills required.
version: 0.1.0
author: GrowthLab (growthlab.mx)
license: MIT
---

# design-md

> GrowthLab-original content. MIT licensed (see repo `LICENSE`).

> **Purpose:** Turn a brand identity manual into `design.md` — a compact, checkable rules file that any LLM reads before it makes anything. Works from a PDF, a website, loose images, or a verbal description.

**Dependencies: none.** This skill does not call other skills. All frameworks are inline.

---

## Why this works

LLMs improvise when context is missing. "Make it on-brand" is unfalsifiable — the model cannot check its own output against it, so it guesses, and the guess is usually a plausible-looking near-miss: navy instead of the actual blue, Montserrat instead of the actual typeface, a logo redrawn from memory.

`design.md` replaces the vibe with rules a model can verify itself against:

| Instead of | You get |
|---|---|
| "use our blue" | `#0B2A4A` — and every other blue is a violation |
| "our brand font" | `Söhne` (licensed, not installed) → renders as `Inter` |
| "keep the logo clean" | clear space = 1× the isotype height; never rotate |
| "does this look on-brand?" | a checklist with falsifiable items |

The failure mode this prevents is not ugliness. It is *confident wrongness* — output that looks finished and is subtly, uncorrectably off.

---

## Inputs

Accept whatever the user has. In descending order of quality:

1. **Brand manual PDF** — most common. Best source. Often vectorized (see Step 2).
2. **Live website** — CSS custom properties and computed styles are authoritative for digital.
3. **Loose images / screenshots** — lowest fidelity; colors are compressed and unreliable. Say so.
4. **Verbal description only** — build a provisional `design.md`, mark every value `UNVERIFIED`, and ask for the manual.

Also ask for, if they exist:
- The **source logo files** (`.svg`, `.ai`, `.eps`, `.pdf`) — you will need these; see Step 3.
- The **font files** (`.otf`, `.ttf`) or the licence they hold.

If the user has more than one source, reconcile them and record which won. A printed manual and a live site frequently disagree. The manual is authoritative for print, the site for digital — say which, do not silently average them.

---

## Process

### Step 0 — Ingest

Identify what you have and set expectations out loud:

- PDF → go to Step 1 (colors) and Step 2 (typography). Both are mechanical. Do them with code.
- Website → read the CSS. Do not screenshot the site and eyedrop it.
- Images → warn the user that JPEG/PNG-from-JPEG colors are lossy, then sample anyway and mark values `APPROX`.
- Verbal → mark everything `UNVERIFIED`.

Never proceed on a guess without labelling it as one.

---

### Step 1 — Extract colors (exactly, never by eye)

**"Looks navy" is a failure.** So is reading a hex off a rendered screenshot without checking it against the vector source. Get the real number.

**Order of authority — use the highest available:**

1. **Hex printed as text in the manual.** If the PDF has a text layer and the page literally says `#0B2A4A`, that is the answer. Regex the extracted text for `#[0-9A-Fa-f]{6}`.
2. **Vector fill colors.** `page.get_drawings()` returns the exact fill of every filled path — including the color swatches. This is the source value, not a rendering of it.
3. **Pixel sample of a rendered page.** Render to PNG at ~1400px wide and read pixels. Use only when 1 and 2 are unavailable (raster-only pages).
4. **Eyedropping a screenshot.** Never.

**This ordering is not pedantry.** Rendering quantizes. A PDF stores fills as floats; the renderer converts them to bytes, and that conversion does not always agree with `round(f × 255)`. The two agree only when the stored float is exactly *n*/255 — which real manuals rarely produce, because their colors originate as CMYK, or as truncated decimals from a design tool.

Measured on a two-swatch page whose vector fills were `#0B2A4A` and `#FF6B35`, pixel-sampling the rendered PNG returned `#0A2A49` and `#FF6B35`. The accent round-tripped exactly; the primary came back **two channels off**. Nothing on the page tells you which swatch is which. An off-by-one hex survives every review and is then permanently wrong in every deliverable downstream.

Rendered pages also produce anti-aliasing fringes: a page with two flat swatches, one stroked outline, and two lines of text rendered to **273 distinct colors**. When you must sample pixels, count frequencies and keep only high-frequency colors from swatch interiors — never sample near an edge.

One more trap: `get_drawings()` hands you a 3-component RGB triple **even when the PDF specified the fill in CMYK** (a CMYK fill of 96/72/38/28 comes back as `#124262`). That RGB is a conversion, not the source value. Never back-convert it to CMYK for the manual — read CMYK from the text.

Runnable snippets for all of the above: `references/extraction-cookbook.md`.

**Also capture, verbatim, do not convert:**
- CMYK / Pantone / RAL values if the manual states them. Hex↔CMYK conversion is device-dependent and lossy; transcribe what the manual says and note the conversion was *not* performed.
- Tints and shades if the system defines them (e.g. `Blue 40%`), as their own tokens.

**Every color needs a role.** A hex with no stated job is an invitation to improvise. See the Palette schema below.

---

### Step 2 — Extract typography (the part everyone gets wrong)

**The trap:** brand manuals are usually exported with text converted to outlines. `page.get_text()` returns `""`. It looks like the PDF has no typography information at all. Most people give up here and guess the typeface from the shapes.

**The technique:** the fonts are still *embedded in the PDF* even when no text is extractable. Read the font table directly.

```python
import re
import pymupdf as fitz          # PyMuPDF; `import fitz` also works

doc = fitz.open("brand-manual.pdf")
seen = {}

for pno in range(doc.page_count):
    page = doc[pno]
    vectorized = not page.get_text().strip()   # no text layer? fonts are still listed.
    for xref, ext, ftype, basefont, name, encoding, _ref in page.get_fonts(full=True):
        # Embedded subsets are prefixed with six uppercase letters + '+':
        #   'ABCDEE+FoundersGrotesk-Bold' -> 'FoundersGrotesk-Bold'
        family = re.sub(r"^[A-Z]{6}\+", "", basefont)
        seen.setdefault(family, {
            "type": ftype,                     # Type1 / TrueType / Type0 ...
            "embedded": ext != "n/a",          # False => relies on the reader's system font
            "pages": [],
        })["pages"].append(pno + 1)
    if vectorized:
        print(f"page {pno+1}: text outlined — get_fonts() is your only source")

for family, meta in sorted(seen.items()):
    print(f"{family:32} {meta['type']:9} embedded={meta['embedded']} pages={meta['pages'][:6]}")
```

`get_fonts(full=True)` yields `(xref, ext, type, basefont, name, encoding, referencer)`. Two things matter:

- **`basefont`** — the real typeface name, e.g. `ABCDEE+FoundersGrotesk-Bold`. Strip the six-letter subset prefix.
- **`ext`** — `"n/a"` means the font is *not* embedded (the PDF expects the reader to have it). Any other value (`ttf`, `otf`, `cff`) means the font file is inside the PDF and you can extract it with `doc.extract_font(xref)`.

No text layer required. This works on fully-outlined manuals.

> No Python? `pdffonts brand-manual.pdf` (poppler-utils) prints the same table.

**Then classify each family:**

Is it commercially licensed, or is it in the open Google Fonts catalog? Check against the catalog — **do not trust the `fonts.googleapis.com` status code.** Google serves some commercially-licensed faces from that endpoint too: `css2?family=Proxima+Nova`, `Helvetica+Neue`, and `Avenir` all return **HTTP 200**, and none of them are open fonts.

Two checks that actually work (verified to agree across 9 test families):

- **Ground truth** — `https://raw.githubusercontent.com/google/fonts/main/{ofl,apache,ufl}/<slug>/METADATA.pb` returns 200 only for the open catalog, and the directory tells you the licence. `<slug>` is the family name lowercased with all non-alphanumerics removed (`EB Garamond` → `ebgaramond`).
- **Fast heuristic** — fetch `css2?family=<Name>` and look at the `src` URL. Open-catalog fonts serve from `fonts.gstatic.com/s/…`; restricted ones from `fonts.gstatic.com/l/font?kit=…`. The `/l/` path means *licensed, not yours to use*.

**Then pick the fallback, and document both.**

If the real typeface is commercial and not installed on the machine that will render the output, the output *will not use it*. Pretending otherwise is the lie that makes `design.md` worthless. So:

- Write the stack **real font first, fallback second, generic last**: `"Söhne", "Inter", sans-serif`.
- State plainly which one is actually installed and therefore actually rendering.
- Verify installation rather than assuming — see the cookbook for a cross-platform check.

**Metric-compatible vs. stylistic substitutes.** A *metric-compatible* substitute has identical advance widths, so line breaks and page counts match exactly. A *stylistic* substitute merely looks similar and will reflow. Know which you're using:

| Commercial original | Metric-compatible (identical widths) | Licence |
|---|---|---|
| Arial | Arimo, Liberation Sans | OFL |
| Times New Roman | Tinos, Liberation Serif | OFL |
| Courier New | Cousine, Liberation Mono | OFL |
| Calibri | Carlito | OFL |
| Cambria | Caladea | OFL |

| Commercial original | Stylistic substitute (will reflow) | Licence |
|---|---|---|
| Helvetica / Helvetica Neue | Inter, Arimo | OFL |
| Futura | Jost, Questrial | OFL |
| Avenir / Avenir Next | Nunito Sans, Manrope | OFL |
| Proxima Nova | Montserrat, Inter | OFL |
| Franklin Gothic | Libre Franklin | OFL |
| Akzidenz-Grotesk | Archivo, Inter | OFL |
| DIN / DIN Next | Barlow, Archivo, Chivo | OFL |
| DIN Condensed | Oswald, Archivo Narrow | OFL |
| Garamond | EB Garamond, Cormorant Garamond | OFL |
| Baskerville | Libre Baskerville | OFL |
| Caslon | Libre Caslon Text | OFL |
| Bodoni / Didot | Libre Bodoni, Playfair Display | OFL |

*(Every family in the right-hand columns was verified present in the open Google Fonts catalog under OFL. Substitution is a rendering convenience, not a licence to imitate a protected design in a new logotype.)*

**Finally, the type scale.** Derive it from the manual — sizes, weights, and what each level is *for*. Do not invent a scale. If the manual genuinely omits one, say so, propose a modular scale (e.g. base 16px × 1.25 major third), and mark it `PROPOSED`.

---

### Step 3 — Logo & isotype

**The hard rule: never hand-redraw a logo. Extract the real vector.**

Not "trace it carefully." Not "recreate it in SVG, it's just a circle and a wordmark." Extract it.

Why this is non-negotiable:

- A redrawn mark is **subtly wrong and everyone notices.** Not consciously, and not immediately — but the curve tension, the joint radii, the optical corrections, the letterspacing, the exact terminal angles *are* the identity. Reproducing them from observation gets you 97% of the way there, and the missing 3% is the entire reason the brand paid for the mark. It reads as counterfeit without the viewer being able to say why.
- A logo is usually a **registered trademark.** An approximation is not a safer legal position than a copy — it is a worse one, because now it is both unauthorized *and* a distortion of the mark the owner is obliged to police.
- It is **unnecessary.** The real vector nearly always exists and takes minutes to obtain.

Where to get the real vector, in order:

1. **Ask for the source file** — `.svg`, `.ai`, `.eps`. Always try this first.
2. **From the website** — inline `<svg>` elements (copy `outerHTML`) or `<img src="*.svg">` (download it).
3. **From the PDF** — crop the page to the logo's bounding box and export vector SVG (`page.get_svg_image()`), or use `pdftocairo -svg` / `mutool draw -F svg`. Snippets in the cookbook.
4. **Raster only?** Then the logo is raster. Record the PNG path, note the constraint, and ask for the vector. Do not "upgrade" it by tracing.

If you cannot obtain a real vector, `design.md` must say `LOGO VECTOR UNAVAILABLE — do not reproduce the mark; place the supplied raster only.` That is a better outcome than a plausible forgery.

**Capture the construction rules and the prohibitions.** Manuals state these explicitly; transcribe them, don't paraphrase them into vagueness:

- Minimum clear space (usually expressed as a multiple of some element of the mark).
- Minimum size, print and screen.
- Approved lockups (horizontal, stacked, isotype-only) and when each is used.
- Approved color versions (full color, mono, reversed/knockout).
- **Hard prohibitions** — the "never" list. Never rotate, never stretch, never recolor, never add effects, never place on busy backgrounds, never enclose in a shape, never alter spacing between isotype and wordmark. Copy the manual's actual list; add nothing it doesn't say.

Record **canonical file paths** for every approved asset. A rule pointing at nothing is not a rule.

---

### Step 4 — Voice

Shorter than the visual sections, and routinely skipped. Don't skip it — it is where an LLM improvises most freely.

Extract:

- **Tagline** — verbatim, with any trademark/punctuation exactly as written.
- **Tone keywords** — 3–6 adjectives. If the manual gives contrasts ("confident, not arrogant"), keep the contrast; it carries the information.
- **Signature phrases** — things the brand actually says, spelled the way the brand spells them (product names, capitalization, hyphenation).
- **Banned words** — what the brand *never* says. This is the highest-value field in the whole voice section and the one manuals most often omit. If it's absent, ask. "What words make you wince?" gets a better answer than "what is your tone?"
- **Grammatical conventions** — Oxford comma, sentence case vs. title case in headings, first person plural vs. singular, contractions allowed or not.

---

### Step 5 — Structure the output

**The schema is the product.** Emit `design.md` with exactly these sections, in this order. A worked, neutral example: `references/design-md-template.md`.

Open with a short provenance header — source file, date extracted, and which source won any conflicts.

#### `## Palette`

A table, plus rules. The table is not enough on its own; a bare list of hexes tells a model *what exists*, not *what to do*.

| Token | Hex | Role |
|---|---|---|
| `ink` | `#0B2A4A` | Primary. Body text, headings, primary surfaces. |
| `signal` | `#FF6B35` | Accent only. CTAs and emphasis. Never a background. |

Then, explicitly:
- Which color **must appear on every deliverable**.
- Which are **accents only**, with a ceiling if the manual sets one ("≤10% of surface area").
- **Forbidden colors** — including near-misses of the brand colors, and any color the manual bans outright.
- Tints/shades, if defined, as their own tokens.

#### `## Typography`

Primary and Secondary, with real names, licensing status, and the render fallback stack:

| Role | Real typeface | Licence | Renders as | Stack |
|---|---|---|---|---|
| Primary | Söhne | Commercial (not installed) | Inter | `"Söhne", "Inter", sans-serif` |

Then the type scale — Display / H1 / H2 / Body / Caption — with **size + weight + usage** for each. Usage is the column that stops improvisation.

#### `## Logo & Isotype`

Construction rules, canonical file paths, and the hard "never" list from Step 3.

#### `## Voice`

Tagline, tone keywords, writing rules, banned words.

#### `## Do / Don't`

A scannable two-column table. **This is what stops an LLM from improvising** — it converts the preceding sections into paired, concrete instructions.

| Do | Don't |
|---|---|
| Use `#0B2A4A` for all body text | Use pure black `#000000` |
| Set headings in the Primary stack | Substitute a "similar" geometric sans |
| Place the supplied `logo-full.svg` | Redraw, trace, or recolor the mark |

Every row must be checkable by looking at the output. "Do: be bold" is not a row. "Don't: use more than two accent colors per layout" is.

#### `## Deliverable checklist`

A checkbox list the model verifies against its own output *before shipping anything*.

```markdown
- [ ] Every color used appears in the Palette table
- [ ] `ink` (#0B2A4A) appears on the deliverable
- [ ] Accent coverage ≤ 10% of surface area
- [ ] All type is set in the Primary or Secondary stack
- [ ] Logo is the supplied `assets/logo-full.svg` — not redrawn
- [ ] Clear space around the logo ≥ 1× isotype height
- [ ] No banned word appears in the copy
```

---

### Step 6 — Verify

Do not hand over `design.md` until it passes this self-check. State the result of each item; do not just assert "verified."

| Check | Fails when |
|---|---|
| **Every hex has a role** | A color appears in the Palette table with an empty or vague Role cell |
| **Every hex is well-formed** | Not `#RRGGBB`, uppercase, 6 digits (normalize 3-digit and `rgb()` forms) |
| **No orphan colors** | A hex appears in Do/Don't, Typography, or the checklist but not in Palette |
| **No font claimed that isn't installed** | Any typeface listed as rendering without an installation check having been run |
| **Fallbacks are real** | Each fallback verified present in the open Google Fonts catalog (or otherwise obtainable) |
| **Logo paths resolve** | A canonical path in `## Logo & Isotype` does not exist on disk |
| **Checklist is falsifiable** | Fewer than one item that could be mechanically checked against a deliverable |
| **Provenance is recorded** | A value cannot be traced to a source (PDF page N, CSS variable, user statement) |
| **Guesses are labelled** | Any `APPROX` / `UNVERIFIED` / `PROPOSED` value is unmarked |

The orphan-color check is worth automating — it catches the most common regression (someone edits Do/Don't and introduces a hex that was never in the palette). A script for it is in `references/extraction-cookbook.md`.

If a check fails, fix it or mark the value. **Never resolve a failed check by deleting the check.**

---

## How to use `design.md`

Put it where the agent will actually read it:

- **Project root** — `design.md`. Simplest, most discoverable.
- **`.agents/design.md`** — if you keep agent context in one place.
- **Referenced from `CLAUDE.md`** — add a line: `Before any creative task, read design.md and follow it exactly.`

Then make reading it **step 1 of every creative task**. Not a suggestion in the middle of a prompt — the first instruction. The file only works if it is loaded before the model starts generating, because a model cannot retroactively un-improvise.

Two habits that keep it honest:

1. **Re-run the Step 6 verification whenever `design.md` changes.** It rots the same way code does.
2. **When the model violates a rule, don't just correct the output — ask whether the rule was checkable.** Most violations trace back to an unfalsifiable row in Do/Don't.

---

## References & credits

**No third-party skill, framework, or copyrighted text is reproduced here.** The process and schema above are GrowthLab-original, MIT licensed.

External tools referenced by the cookbook (invoked by the user; not vendored into this repo):

| Tool | Purpose | Licence |
|---|---|---|
| [PyMuPDF](https://pymupdf.readthedocs.io/) (Artifex) | PDF font tables, vector fills, rendering, SVG export | AGPL-3.0 or commercial |
| [Poppler](https://poppler.freedesktop.org/) (`pdffonts`, `pdftoppm`, `pdftocairo`) | Same, as CLI | GPL-2.0 |
| [fontTools](https://github.com/fonttools/fonttools) | Reading font family names | MIT |
| [MuPDF](https://mupdf.com/) (`mutool`) | PDF → SVG | AGPL-3.0 or commercial |
| [Pillow](https://python-pillow.org/) | Optional raster sampling | MIT-CMU |

> **Note on PyMuPDF:** it is AGPL-3.0 (or a paid commercial licence from Artifex). Running it as a local extraction tool is unproblematic, but if you embed it in a networked service you must satisfy the AGPL or buy a licence. MIT-licensed alternatives for the parts that need them: `pypdf` (BSD), `pdfplumber` (MIT), `pdfminer.six` (MIT). The `get_fonts()` technique in Step 2 is PyMuPDF-specific; the closest MIT equivalent is walking `/Resources /Font` in the page dictionary with `pypdf`, or shelling out to `pdffonts`.

Font names in the substitution tables are trademarks of their respective owners and are listed for identification only. Metric-compatible pairs are documented behaviour of the Liberation and Croscore font projects.

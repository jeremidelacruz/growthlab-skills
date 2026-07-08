# Extraction cookbook

Runnable snippets for the `design-md` skill. Every snippet here was executed against a real PDF before being written down.

> GrowthLab-original content. MIT licensed (see repo `LICENSE`).

**Setup**

```bash
pip install pymupdf          # provides both `import pymupdf` and `import fitz`
# optional
pip install fonttools        # nicer font-family parsing
# CLI alternatives (no Python): poppler-utils gives you pdffonts / pdftoppm / pdftocairo
```

Throughout: `import pymupdf as fitz` is the modern import. Plain `import fitz` still works.

---

## 1. Read embedded font names from a PDF

**The single most valuable technique in this skill.** Works when the manual is fully vectorized and `get_text()` returns nothing.

```python
import re
import pymupdf as fitz

def pdf_fonts(path):
    doc = fitz.open(path)
    seen = {}
    for pno in range(doc.page_count):
        page = doc[pno]
        outlined = not page.get_text().strip()
        for xref, ext, ftype, basefont, name, encoding, _ref in page.get_fonts(full=True):
            family = re.sub(r"^[A-Z]{6}\+", "", basefont)   # strip subset prefix
            rec = seen.setdefault(family, {
                "xref": xref,
                "type": ftype,          # Type1 | TrueType | Type0 | ...
                "embedded": ext != "n/a",
                "ext": ext,
                "pages": [],
            })
            rec["pages"].append(pno + 1)
        if outlined:
            print(f"  page {pno+1}: text outlined — get_fonts() is the only source")
    return seen

for family, m in sorted(pdf_fonts("brand-manual.pdf").items()):
    flag = "embedded" if m["embedded"] else "NOT embedded (needs system font)"
    print(f"{family:34} {m['type']:9} {flag:32} pages={m['pages'][:6]}")
```

`get_fonts(full=True)` returns tuples of
`(xref, ext, type, basefont, name, encoding, referencer)`.

- `basefont` — the real name, e.g. `ABCDEE+FoundersGrotesk-Bold`. The six uppercase letters + `+` mark an embedded **subset**; strip them.
- `ext` — `"n/a"` means not embedded. Anything else (`ttf`, `otf`, `cff`) means the font file is inside the PDF.

**Dump the actual font file** (when embedded), so you can install it or read its metadata:

```python
doc = fitz.open("brand-manual.pdf")
basename, ext, subtype, buf = doc.extract_font(xref)      # xref from above
if buf:
    with open(f"{basename}.{ext}", "wb") as fh:
        fh.write(buf)
```

Pass `info_only=True` to inspect without extracting bytes.

**No Python:**

```bash
pdffonts brand-manual.pdf     # poppler-utils; prints name, type, emb, sub, uni, object ID
```

---

## 2. Render pages to PNG (~1400px wide)

```python
import pymupdf as fitz

def render(path, out_dir=".", target_width=1400):
    doc = fitz.open(path)
    for pno in range(doc.page_count):
        page = doc[pno]
        zoom = target_width / page.rect.width
        pix = page.get_pixmap(matrix=fitz.Matrix(zoom, zoom))   # RGB, no alpha
        pix.save(f"{out_dir}/page-{pno+1:02d}.png")
        print(f"page {pno+1}: {pix.width}x{pix.height}")
```

`dpi=` is an alternative to `matrix=` (`page.get_pixmap(dpi=150)`), but targeting a pixel width keeps output consistent across mixed page sizes.

**No Python:** `pdftoppm -png -r 150 brand-manual.pdf page`

---

## 3. Colors — exact, in order of authority

### 3a. Hex printed as text (highest authority)

```python
import re, pymupdf as fitz

doc = fitz.open("brand-manual.pdf")
for pno in range(doc.page_count):
    for hx in re.findall(r"#[0-9A-Fa-f]{6}\b", doc[pno].get_text()):
        print(f"page {pno+1}: {hx.upper()}")
```

Also worth grepping for `CMYK`, `RGB`, `Pantone`, `PMS`, `RAL` lines nearby — transcribe them verbatim, never convert.

### 3b. Vector fill colors (authoritative for swatches)

```python
import pymupdf as fitz

def hexof(triple):
    return "#%02X%02X%02X" % tuple(round(c * 255) for c in triple)

doc = fitz.open("brand-manual.pdf")
for pno in range(doc.page_count):
    for d in doc[pno].get_drawings():
        # d["type"]: 'f' filled, 's' stroked, 'fs' both
        if d.get("fill"):
            r = d["rect"]
            area = abs(r.width * r.height)
            if area > 500:                     # ignore hairlines and tiny glyph paths
                print(f"page {pno+1}  fill={hexof(d['fill'])}  area={area:.0f}  {r}")
        if d.get("color"):
            print(f"page {pno+1}  stroke={hexof(d['color'])}")
```

PyMuPDF normalizes every fill to a **3-component RGB float triple in `0.0–1.0` — even when the PDF specified the fill in CMYK.** Verified: a CMYK fill of 96/72/38/28 comes back as `#124262`, with no indication it was ever CMYK. That RGB is a conversion, not the source value. If the manual quotes CMYK or Pantone, take those from the text (3a); never back-convert from these RGB values.

### 3c. Pixel sampling (fallback only — raster pages)

**Rendering quantizes.** A PDF stores fills as floats, and the renderer's float→byte conversion does not always agree with `round(f × 255)`. They agree only when the stored float is exactly *n*/255 — which real manuals rarely produce, since their colors originate as CMYK or as truncated design-tool decimals.

Measured on a page whose vector fills were `#0B2A4A` and `#FF6B35`, pixel-sampling returned `#0A2A49` and `#FF6B35`. The accent round-tripped exactly; the primary came back two channels off — and nothing distinguishes them by eye. Prefer 3b whenever the swatch is vector.

Anti-aliasing is the other hazard: a page with two flat swatches, one stroked outline, and two lines of text rendered to **273 distinct colors**. Count frequencies, keep the peaks, and sample swatch interiors only.

```python
from collections import Counter
import pymupdf as fitz

page = fitz.open("brand-manual.pdf")[0]
zoom = 1400 / page.rect.width
pix = page.get_pixmap(matrix=fitz.Matrix(zoom, zoom))
assert pix.n == 3 and pix.stride == pix.width * pix.n     # RGB, no padding

# Dominant colors across the page (~0.3s for a 1400px A4 page)
data = pix.samples
counts = Counter(data[i:i+3] for i in range(0, len(data), 3))
total = pix.width * pix.height
for px, c in counts.most_common(12):
    if c / total > 0.005:                                  # drop AA fringes
        print("#%02X%02X%02X" % tuple(px), f"{100*c/total:.1f}%")

# A single, deliberate sample from inside a known swatch
print("#%02X%02X%02X" % pix.pixel(300, 300))               # pix.pixel(x, y) -> (r, g, b)
```

For images the user hands you directly (screenshots, JPEGs), the same frequency approach works via Pillow — but mark every value `APPROX`, because JPEG chroma subsampling has already moved the colors.

---

## 4. Pull colors, fonts, and the logo from a live website

### 4a. CSS custom properties (browser console / headless)

Same-origin stylesheets only; cross-origin `.cssRules` access throws.

```js
const vars = {};
for (const sheet of document.styleSheets) {
  let rules;
  try { rules = sheet.cssRules; } catch { continue; }   // cross-origin — fetch separately
  for (const rule of rules ?? []) {
    if (!rule.style) continue;
    for (const prop of rule.style) {
      if (prop.startsWith('--')) vars[prop] = rule.style.getPropertyValue(prop).trim();
    }
  }
}
console.table(vars);
```

Resolved values on the root element (catches runtime theme overrides):

```js
const cs = getComputedStyle(document.documentElement);
Object.fromEntries(Object.keys(vars).map(v => [v, cs.getPropertyValue(v).trim()]));
```

### 4b. Fonts actually in use

```js
// Declared families, by element type
[...new Set(['body','h1','h2','h3','p','a','button']
  .flatMap(sel => [...document.querySelectorAll(sel)])
  .map(el => getComputedStyle(el).fontFamily))]

// Fonts the page actually loaded, with weights
[...document.fonts].map(f => ({ family: f.family, weight: f.weight, style: f.style, status: f.status }))

// @font-face sources (where the real files live)
[...document.styleSheets].flatMap(s => { try { return [...s.cssRules]; } catch { return []; } })
  .filter(r => r instanceof CSSFontFaceRule)
  .map(r => ({ family: r.style.getPropertyValue('font-family'), src: r.style.getPropertyValue('src') }))
```

### 4c. The logo — extract the real vector, never redraw

```js
// Inline SVGs, largest first — the logo is usually among them
[...document.querySelectorAll('svg')]
  .map(s => ({ w: s.getBoundingClientRect().width, cls: s.getAttribute('class'), svg: s.outerHTML }))
  .sort((a, b) => b.w - a.w)
  .slice(0, 5);

// Linked SVG assets
[...document.querySelectorAll('img[src$=".svg"], img[srcset*=".svg"]')].map(i => i.src);
```

Watch for `<use xlink:href="#id">` — the geometry lives in a sprite elsewhere in the DOM; grab that `<symbol>` too, or the copied SVG will be empty.

### 4d. No browser: stdlib only

```python
import re, urllib.request
from urllib.parse import urljoin

UA = {"User-Agent": "Mozilla/5.0"}
def get(url):
    with urllib.request.urlopen(urllib.request.Request(url, headers=UA), timeout=15) as r:
        return r.read().decode("utf-8", "replace")

def site_tokens(url):
    html = get(url)
    sheets = [urljoin(url, h) for h in re.findall(r'<link[^>]+rel=["\']stylesheet["\'][^>]+href=["\']([^"\']+)', html)]
    css = html + "".join(get(s) for s in sheets)
    colors = dict(re.findall(r"(--[\w-]+)\s*:\s*(#[0-9A-Fa-f]{3,8}|rgba?\([^)]+\)|hsla?\([^)]+\))", css))
    faces  = re.findall(r"font-family\s*:\s*([^;}]+)", css)
    svgs   = [urljoin(url, s) for s in re.findall(r'src=["\']([^"\']+\.svg)', html)]
    return colors, sorted(set(f.strip() for f in faces)), svgs
```

Note that `<link>` tags with attributes in the reverse order (`href` before `rel`) will be missed by that regex — for anything load-bearing, use a real parser or the browser console.

---

## 5. Is this font commercial, or open?

**Do not use the `fonts.googleapis.com` status code as your test.** Google serves some commercially-licensed faces from that endpoint. Verified: `css2?family=Proxima+Nova`, `Helvetica+Neue`, and `Avenir` all return **HTTP 200**, and none are open fonts. `Futura` correctly returns 400 — so the check *looks* like it works until it silently doesn't.

Two checks that hold up (verified in agreement across 9 families):

```python
import re, urllib.request, urllib.error
UA = {"User-Agent": "Mozilla/5.0"}

def _status(url):
    try:
        with urllib.request.urlopen(urllib.request.Request(url, headers=UA), timeout=15) as r:
            return r.status, r.read().decode("utf-8", "replace")
    except urllib.error.HTTPError as e:
        return e.code, ""
    except Exception:
        return 0, ""

def in_open_catalog(family):
    """Ground truth: the google/fonts repo. Returns 'OFL' | 'APACHE' | 'UFL' | None."""
    slug = re.sub(r"[^a-z0-9]", "", family.lower())        # 'EB Garamond' -> 'ebgaramond'
    for lic in ("ofl", "apache", "ufl"):
        code, _ = _status(f"https://raw.githubusercontent.com/google/fonts/main/{lic}/{slug}/METADATA.pb")
        if code == 200:
            return lic.upper()
    return None

def gstatic_path(family):
    """Fast heuristic: open fonts serve from /s/, restricted ones from /l/."""
    code, body = _status("https://fonts.googleapis.com/css2?family=" + family.replace(" ", "+"))
    if code != 200:
        return "not served"
    dirs = set(re.findall(r"https://fonts\.gstatic\.com/(\w+)/", body))
    if "l" in dirs:
        return "restricted (/l/) — licensed, not open"
    return "open (/s/)" if "s" in dirs else "unknown"

for f in ["Inter", "Montserrat", "Proxima Nova", "Helvetica Neue", "Avenir", "Futura"]:
    print(f"{f:18} catalog={str(in_open_catalog(f)):6} {gstatic_path(f)}")
```

Expected output (verified):

```
Inter              catalog=OFL    open (/s/)
Montserrat         catalog=OFL    open (/s/)
Proxima Nova       catalog=None   restricted (/l/) — licensed, not open
Helvetica Neue     catalog=None   restricted (/l/) — licensed, not open
Avenir             catalog=None   restricted (/l/) — licensed, not open
Futura             catalog=None   not served
```

The two columns agree on every family: `catalog=None` always coincides with `/l/` or "not served". Trust the catalog column; the `/l/` heuristic is the cheap single-request version.

---

## 6. Is the font actually installed on this machine?

If it isn't installed, it isn't rendering — no matter what the font stack says. Check, don't assume. Zero extra dependencies (reuses PyMuPDF):

```python
import os, sys, glob
import pymupdf as fitz

def installed_families():
    if sys.platform == "win32":
        dirs = [os.path.join(os.environ["WINDIR"], "Fonts"),
                os.path.join(os.environ.get("LOCALAPPDATA", ""), r"Microsoft\Windows\Fonts")]
    elif sys.platform == "darwin":
        dirs = ["/System/Library/Fonts", "/Library/Fonts", os.path.expanduser("~/Library/Fonts")]
    else:
        dirs = ["/usr/share/fonts", "/usr/local/share/fonts",
                os.path.expanduser("~/.fonts"), os.path.expanduser("~/.local/share/fonts")]

    families = set()
    for d in filter(lambda p: p and os.path.isdir(p), dirs):
        for ext in ("ttf", "otf", "ttc"):
            for path in glob.glob(os.path.join(d, "**", f"*.{ext}"), recursive=True):
                try:
                    families.add(fitz.Font(fontfile=path).name)   # e.g. 'Arial Bold'
                except Exception:
                    pass
    return families

fams = installed_families()
for probe in ["Söhne", "Inter", "Arial"]:
    hits = [f for f in fams if probe.lower() in f.lower()]
    print(f"{probe:10} installed={bool(hits):<5} {hits[:2]}")
```

`fitz.Font(...).name` returns family + subfamily merged (`"Arial Bold"`), so match by substring.

**Alternatives:** `fc-list : family` (Linux/macOS with fontconfig); `system_profiler SPFontsDataType` (macOS); or `fontTools.ttLib.TTFont(path)['name']` for precise name-ID 1/16 records.

---

## 7. Extract the logo vector from a PDF

Crop the page to the logo's bounding box, then export SVG. Get the bbox from `get_drawings()` (see 3b) or from `page.get_image_info()` for raster logos.

```python
import pymupdf as fitz

doc  = fitz.open("brand-manual.pdf")
page = doc[2]                                    # the page showing the logo

page.set_cropbox(fitz.Rect(40, 40, 210, 210))    # logo bbox in PDF points
svg = page.get_svg_image(text_as_path=True)      # text_as_path keeps letterforms exact
open("logo.svg", "w", encoding="utf-8").write(svg)
```

Set `text_as_path=True` for a logotype: it preserves the exact letterforms instead of emitting a `<text>` element that depends on a font the viewer may not have.

If the logo is a **raster** image embedded in the PDF, pull the original bytes rather than re-rendering:

```python
for xref, *_ in page.get_images(full=True):
    img = doc.extract_image(xref)                # {'image': bytes, 'ext': 'png', ...}
    open(f"logo-{xref}.{img['ext']}", "wb").write(img["image"])
```

**No Python:** `pdftocairo -svg -f 3 -l 3 brand-manual.pdf logo.svg` or `mutool draw -F svg -o logo.svg brand-manual.pdf 3`

A raster logo stays a raster logo. Do not trace it. See the skill's Step 3.

---

## 8. Verify `design.md` — the orphan-color check

The most common regression: someone edits the Do/Don't table and introduces a hex that was never declared in the Palette. Automate it.

```python
import re, sys

def check(path="design.md"):
    md = open(path, encoding="utf-8").read()
    errors = []

    # Palette section = from '## Palette' to the next '## '
    m = re.search(r"^## Palette\s*$(.*?)(?=^## )", md, re.S | re.M)
    if not m:
        return ["no '## Palette' section found"]
    palette_body = m.group(1)

    declared = set(h.upper() for h in re.findall(r"#[0-9A-Fa-f]{6}\b", palette_body))
    used     = set(h.upper() for h in re.findall(r"#[0-9A-Fa-f]{6}\b", md))

    for orphan in sorted(used - declared):
        errors.append(f"orphan color {orphan}: used but not declared in ## Palette")

    # Every palette row must have a non-empty Role cell.
    # Strip backticks: the template writes hexes as `#RRGGBB`.
    for row in re.findall(r"^\|(.+)\|\s*$", palette_body, re.M):
        cells = [c.strip().strip("`").strip() for c in row.split("|")]
        if len(cells) >= 3 and re.match(r"^#[0-9A-Fa-f]{6}$", cells[1] or ""):
            if not cells[2] or cells[2] in {"-", "TBD", "?"}:
                errors.append(f"color {cells[1]} has no Role")

    # Malformed / shorthand hex
    for bad in re.findall(r"#[0-9A-Fa-f]{3}\b(?![0-9A-Fa-f])", md):
        errors.append(f"shorthand hex {bad}: expand to 6 digits")

    # Checklist must contain at least one falsifiable item
    if len(re.findall(r"^\s*-\s*\[ \]", md, re.M)) < 1:
        errors.append("deliverable checklist has no checkbox items")

    # Unlabelled guesses
    for tag in ("APPROX", "UNVERIFIED", "PROPOSED"):
        if tag in md:
            print(f"note: {md.count(tag)}x {tag} — confirm these are intentional")

    return errors

errs = check(sys.argv[1] if len(sys.argv) > 1 else "design.md")
print("\n".join(f"FAIL {e}" for e in errs) if errs else "OK — all checks passed")
sys.exit(1 if errs else 0)
```

Extend it with a logo-path existence check once your `## Logo & Isotype` section has real paths:

```python
for p in re.findall(r"`([^`]+\.(?:svg|png|ai|eps))`", md):
    if not os.path.exists(p):
        errors.append(f"logo path does not resolve: {p}")
```

Run it in CI if `design.md` lives in a repo. A rules file nobody validates decays into a rules file nobody follows.

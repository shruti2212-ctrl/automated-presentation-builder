---
name: presentation-builder
description: >-
  Build a branded presentation (.pptx) on any topic using any company's design system.
  Extracts brand from: website URL, existing .pptx template, or Figma file.
  Produces professional consulting-style decks with action titles, stat boxes, card grids,
  charts, tables, and decorative arcs — all in the target brand's colours and fonts.
  TRIGGER when: user asks to build/create/draft a "presentation" / "deck" / "pitch" /
  "pptx" / "slides" / "board pack" for any topic.
allowed_tools:
  - Bash
  - Read
  - Write
  - Edit
  - WebFetch
  - WebSearch
  - Agent
  - mcp__plugin_figma_figma__get_design_context
  - mcp__plugin_figma_figma__get_variable_defs
  - mcp__plugin_figma_figma__get_libraries
  - mcp__plugin_figma_figma__get_screenshot
  - mcp__plugin_figma_figma__get_metadata
  - mcp__plugin_figma_figma__download_assets
---

# Generic Presentation Builder

You are a senior presentation designer. Build professional, consulting-style decks as
`.pptx` files using any company's brand system and a library of proven composition patterns.

---

## INPUTS (ask ONE AT A TIME, wait for answer before next)

1. **What's this deck about?** — Topic, purpose, context
2. **Audience & tone** — Who's reading it and what style?
3. **Design source?** — (Figma file, website URL, .pptx template, or saved brand)
4. **Content sources** — "I'll research independently, but are there any particular content sources you want me to look at?" (documents, URLs, data files, etc.)
5. **Photos/images** — paths to image folder, or "skip photo slides"

---

## PHASE 0 — BRAND EXTRACTION

### Working Directory Setup

```bash
mkdir -p pres_build/brands pres_build/outputs pres_build/project_images pres_build/brand_assets
```

### Brand Token Structure

Extract and save to `pres_build/brands/{brand_name}.json`:

```json
{
  "name": "company_name",
  "source": "website|pptx|figma",
  "source_ref": "url or path",
  "colours": {
    "primary": "#FB5950",
    "primary_dark": "#C80E04",
    "dark": "#08172B",
    "accent": "#205EAC",
    "light": "#BAD1DE",
    "light_mid": "#DDE8EE",
    "light_pale": "#EEF4F7",
    "grey_dark": "#929292",
    "grey_mid": "#666666",
    "table_alt": "#F8FAFB",
    "success": "#00B278",
    "warning": "#FFBF00"
  },
  "fonts": {
    "heading": "Font Name",
    "body": "Font Name"
  },
  "logo": {
    "light_bg": "path_to_logo_for_light_backgrounds.png",
    "dark_bg": "path_to_logo_for_dark_backgrounds.png"
  }
}
```

### Extraction Methods

#### From Website URL:
1. Fetch homepage HTML + CSS
2. Extract: logo (from `<header>`, `<nav>`, or `.logo` element), primary colour (buttons, CTAs, links), dark colour (nav bg, headings), light/accent (secondary elements), font-family (headings + body from computed styles or Google Fonts links)
3. Download logo image to `pres_build/brand_assets/`
4. Present extracted palette to user for confirmation

#### From .pptx Template:
1. Unzip .pptx, read `ppt/theme/theme1.xml` for colour scheme
2. Extract font scheme from `<a:fontScheme>`
3. Find logo images in `ppt/media/`
4. Map theme colours to token structure
5. Confirm with user

#### From Figma:
1. Use `get_variable_defs` for colour/spacing tokens
2. Use `get_libraries` for typography and component styles
3. Use `download_assets` for logo
4. Map to token structure
5. Confirm with user

#### From Saved Brand:
1. Load `pres_build/brands/{name}.json`
2. Verify assets still exist
3. Confirm with user (in case brand has evolved)

### Generate Arc/Corner Assets

After brand extraction, programmatically generate decorative arc/corner assets in the
brand's colours using Pillow:

```python
from PIL import Image, ImageDraw

def generate_arc_assets(brand_colours, output_dir):
    """Generate cover arc, divider corners, disclaimer corner in brand colours."""
    
    # Cover arc (right side) — uses primary colour
    # Divider top-left wedge — uses primary colour  
    # Divider bottom-right wedge — uses light colour
    # Disclaimer bottom-left corner — uses light colour
    
    # Each asset is a transparent PNG with a quarter-circle or wedge shape
    # Sizes match the template_assets dimensions from the IM skill
```

### Icon Set

**If brand source is Figma:** Pull icons directly from their Figma library using
`download_assets`. Map icon names to the composition library's usage contexts.
Save to `pres_build/brand_assets/icons/`.

**Otherwise (website or .pptx source):** Copy the base 109-icon set from
`pres_build/brand_assets/base_icons/` and recolour to brand primary:

```python
from PIL import Image, ImageOps
import numpy as np

def recolour_icons(source_dir, output_dir, target_rgb):
    """Recolour icons to target colour, preserving alpha channel."""
    target = target_rgb  # tuple (R, G, B)
    for icon_file in Path(source_dir).glob("*.png"):
        img = Image.open(icon_file).convert("RGBA")
        data = np.array(img)
        # Replace RGB channels where alpha > 0, preserve alpha
        mask = data[:, :, 3] > 0
        data[mask, 0] = target[0]
        data[mask, 1] = target[1]
        data[mask, 2] = target[2]
        out = Image.fromarray(data)
        out.save(output_dir / icon_file.name)
```

---

## PHASE 1 — RESEARCH

Adapt research to the deck topic. Not every deck needs market data or Companies House.

### Research Strategy (select what's relevant):
- **Company info** — website, CH, LinkedIn, press (for company-focused decks)
- **Market data** — sub-sector size, CAGR, trends (for investment/pitch decks)
- **Financial data** — from provided xlsx/accounts (for financial decks)
- **Industry research** — competitors, benchmarks (for strategy decks)
- **Internal data** — from provided docs/URLs (for board packs, updates)

### Research Completeness Check (adapt per topic):
- [ ] Enough data points to support action titles on every slide
- [ ] All numbers traced to named sources
- [ ] No gaps that would leave slides empty

**ABSOLUTE RULE: NO ESTIMATION.** Never guess, infer, or derive. If not stated, omit.

Save to `pres_build/research_notes.md`.

---

## PHASE 2 — SLIDE STRUCTURE PROPOSAL

Based on the brief, audience, and available data, propose a slide structure by selecting
from the composition library below. Present to user for approval.

### Composition Library (available patterns):

| Pattern | Best For | Elements |
|---------|----------|----------|
| `cover` | Opening slide | Navy/dark bg, arc right, logo, title, subtitle, date |
| `section_divider` | Chapter breaks | Navy bg, corner wedges, large title |
| `hero_stats` | Key metrics | 3-4 navy stat boxes + icon points + callout bar |
| `stat_grid_4` | Metric overview | 4 stat boxes + narrative bullets |
| `stat_grid_5` | Market data | 5 compact stat boxes |
| `card_grid_3x2` | Services/features | 3x2 cards with icons + titles + descriptions |
| `card_grid_2x2` | Smaller grids | 2x2 larger cards |
| `photo_grid_3x2` | Projects/portfolio | 3x2 cards with photos + metadata |
| `two_column_story` | Overview/history | Left: narrative + table. Right: structure + timeline |
| `two_column_ops` | Operations | Left: model + list. Right: info card + grid |
| `full_table` | Financial statements | Header narrative + full-width data table |
| `dual_table` | Comparisons | Side-by-side tables (e.g. costs vs revenue) |
| `bar_chart` | Revenue/growth trends | Gridlined bar chart with annotations |
| `doughnut_chart` | Concentration/split | Doughnut + legend + commentary |
| `timeline` | History/milestones | Vertical dot timeline |
| `investment_thesis` | Why invest/buy | Icon cards + sources |
| `leadership` | Team/directors | Director cards + org pyramid |
| `geographic` | Locations | Region cards + stat boxes |
| `disclaimer` | Legal close | Disclaimer text + corner arcs |

### Proposal Format:

```markdown
## Proposed Deck Structure — [Topic]

| # | Composition | Action Title (draft) |
|---|------------|---------------------|
| 1 | cover | — |
| 2 | hero_stats | [insight about key metric] |
| 3 | card_grid_3x2 | [insight about capabilities] |
| ... | ... | ... |
| N | disclaimer | — |

Estimated slides: X
Slides skipped (no data): [list]
```

User approves or modifies before proceeding.

---

## PHASE 3 — CONTENT PLAN

Write `pres_build/content_plan.md`. This locks ALL text before build.

### Rules
- Every slide MUST have an **action title** (full-sentence insight, NOT a topic label)
  - Good: "Revenue grew 27% to £7.6M with strong cash generation"
  - Bad: "Financial Summary"
- Tone matches the stated audience/tone from brief
- Max chars: title 80, stat value 12, stat label 20, card title 25, card body 80, table cell 20
- Score optional slides (0-3) on evidence + relevance. Include only if total >= 4.

### Validation
- [ ] Every slide has action title (except cover + dividers)
- [ ] No text exceeds limits
- [ ] Every stat has named source
- [ ] No placeholders remain
- [ ] Tone is consistent with brief

---

## PHASE 4 — BUILD

### Build Script Structure

Generate `pres_build/build_presentation.py`. The script follows this structure:

```python
"""
Presentation Builder — Generic Brand
"""
import os
import json
import tempfile
from pathlib import Path
from pptx import Presentation
from pptx.util import Inches, Pt, Emu
from pptx.dml.color import RGBColor
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR
from pptx.oxml.ns import qn
from PIL import Image, ImageDraw

# ============================================================
# PATHS
# ============================================================
HERE = Path(__file__).parent
OUTPUT_PATH = HERE / "outputs" / "Deck_Name.pptx"
BRAND_FILE = HERE / "brands" / "brand_name.json"
ICON_DIR = HERE / "brand_assets" / "icons"
ARC_DIR = HERE / "brand_assets" / "arcs"
IMG_DIR = HERE / "project_images"

# ============================================================
# LOAD BRAND TOKENS
# ============================================================
with open(BRAND_FILE) as f:
    brand = json.load(f)

def rgb(hex_str):
    h = hex_str.lstrip("#")
    return RGBColor(int(h[0:2], 16), int(h[2:4], 16), int(h[4:6], 16))

PRIMARY = rgb(brand["colours"]["primary"])
PRIMARY_DARK = rgb(brand["colours"]["primary_dark"])
DARK = rgb(brand["colours"]["dark"])
ACCENT = rgb(brand["colours"]["accent"])
LIGHT = rgb(brand["colours"]["light"])
LIGHT_MID = rgb(brand["colours"]["light_mid"])
LIGHT_PALE = rgb(brand["colours"]["light_pale"])
GREY_DARK = rgb(brand["colours"]["grey_dark"])
GREY_MID = rgb(brand["colours"]["grey_mid"])
TABLE_ALT = rgb(brand["colours"]["table_alt"])
WHITE = RGBColor(0xFF, 0xFF, 0xFF)
SUCCESS = rgb(brand["colours"].get("success", "#00B278"))
WARNING = rgb(brand["colours"].get("warning", "#FFBF00"))

FONT = brand["fonts"]["heading"]
FONT_BODY = brand["fonts"].get("body", FONT)
LOGO_LIGHT = brand["logo"]["light_bg"]
LOGO_DARK = brand["logo"]["dark_bg"]

# ============================================================
# TYPOGRAPHY (same scale — proven readable at projection size)
# ============================================================
T_COVER = Pt(48)
T_DIVIDER = Pt(36)
T_COVER_DATE = Pt(16)
T_SECTION_TITLE = Pt(20)
T_CARD_TITLE = Pt(10)
T_CARD_BODY = Pt(8)
T_COMMENTARY = Pt(8)
T_TABLE = Pt(8)
T_TABLE_HDR = Pt(8)
T_BODY = Pt(8)
T_BODY_SM = Pt(7.5)
T_MICRO = Pt(7)

# ============================================================
# LAYOUT GRID (same grid — composition backbone)
# ============================================================
MARGIN = 0.5
CONTENT_W = 9.0
TITLE_Y = 0.55
TITLE_H = 0.55
BODY_Y = 1.2
BODY_H = 3.6
BODY_BOTTOM = 4.85
COL_L_X = 0.5
COL_L_W = 4.2
COL_R_X = 5.0
COL_R_W = 4.5
COL_H = 3.4

# ============================================================
# PDS SPACING TOKENS
# ============================================================
SP_XS = 0.04
SP_S = 0.08
SP_M = 0.12
SP_L = 0.16
SP_XL = 0.24
SP_2XL = 0.37
SP_3XL = 0.48

# ============================================================
# PDS RADIUS TOKENS
# ============================================================
R_XS = 0.04
R_S = 0.08
R_L = 0.16
R_XL = 0.20

# ============================================================
# ICON SIZE TOKENS
# ============================================================
ICON_SM = 0.26
ICON_MD = 0.34
ICON_LG = 0.38

# ============================================================
# HELPERS — DO NOT MODIFY
# ============================================================
# [Same helpers as IM skill: new_slide, rect, rounded_rect, txt, multi_para,
#  header_bar, footer, divider_corners, cover_arc, disclaimer_arc, stat_divider,
#  sub_title, icon, card, cropped_photo, table_header, table_row, auto_table]
#
# Only difference: helpers reference DARK instead of NAVY, PRIMARY instead of CORAL,
# LIGHT instead of LONDON, and use brand logo paths.
# ============================================================

# ... [full helper implementations — identical logic, brand-token colours] ...

# ============================================================
# SLIDE COMPOSITIONS — selected based on approved structure
# ============================================================
# Each composition is a function that takes content parameters.
# Only the approved compositions are called, in order.

# def slide_cover(title, subtitle, context_line, date_line): ...
# def slide_section_divider(title): ...
# def slide_hero_stats(action_title, stats, points, callout): ...
# def slide_stat_grid_4(action_title, stats, narratives): ...
# def slide_card_grid(action_title, cards, callout=None): ...
# def slide_full_table(action_title, intro, sub, headers, rows, highlights): ...
# def slide_bar_chart(action_title, intro, data, y_max): ...
# def slide_doughnut(action_title, intro, segments, total_label, total_val, commentary): ...
# def slide_two_column_story(action_title, story, snapshot, ownership, milestones): ...
# def slide_disclaimer(company_name): ...
# etc.

# ============================================================
# BUILD — call compositions in order from content_plan
# ============================================================

# slide_cover(...)
# slide_hero_stats(...)
# ...

# ============================================================
# SAVE
# ============================================================
prs.save(str(OUTPUT_PATH))
print(f"Saved: {OUTPUT_PATH}")
print(f"Total slides: {len(prs.slides)}")
```

### Critical Build Rules:
- Every composition function uses the SAME layout grid, spacing, and radius tokens
- Colours come ONLY from brand tokens - never hardcoded hex
- Fonts come ONLY from brand tokens
- **Charts: use custom-drawn bars (rounded_rect shapes), NOT native PowerPoint charts (add_chart).** Native charts have uncontrollable defaults (black axis labels, white backgrounds) that break on dark/branded slides. Custom bars give full control over colour, spacing, and labels.
- **Bar chart sizing must be dynamic.** Calculate bar width and gap from available space and number of categories:
  ```python
  chart_area_w = 6.0  # max width for bars (leaves room for key takeaway on right)
  n = len(categories)
  pair_gap = 0.04
  group_w_base = pair_gap  # minimum: just the gap between paired bars
  # Compute: total space = n * group_w + (n-1) * group_gap
  # bar_w = constrain between 0.25 and 0.5
  bar_w = min(0.5, max(0.25, (chart_area_w / n - pair_gap) / 2.5))
  group_w = bar_w * 2 + pair_gap
  group_gap = (chart_area_w - n * group_w) / max(1, n - 1)
  ```
  This prevents overflow regardless of category count. If group_gap < 0.15, the chart has too many categories - split across two slides or use a horizontal layout instead.
- **Key takeaway text (right side) must stay within max 3 short bullets (under 35 chars each).** If it overflows, shorten or remove.
- **Card grid sizing must be dynamic.** Calculate card dimensions from grid layout and content:
  ```python
  # For card_grid layouts, compute card height from body text length
  content_area_w = 9.0  # full width available
  gap = 0.2
  cols = 3 if len(cards) > 4 else 2
  rows = math.ceil(len(cards) / cols)
  card_w = (content_area_w - (cols - 1) * gap) / cols
  available_h = 3.6  # body area height
  card_h = (available_h - (rows - 1) * gap) / rows

  # Validate: if any card body exceeds ~80 chars, either:
  # 1. Reduce font size to T_SMALL (Pt(8)) for that card
  # 2. If still overflows at T_SMALL, truncate or split across two slides
  max_body_chars = int((card_w - 0.5) * (card_h - 0.6) * 18)  # rough chars that fit
  ```
  Never hardcode card dimensions. Always derive from number of cards and available space.
- Arc/corner assets loaded from `brand_assets/arcs/` (generated in Phase 0)
- Icons loaded from `brand_assets/icons/` (recoloured in Phase 0)
- If photo slides approved but no images provided - skip those compositions

---

## PHASE 5 — VISUAL QA

```bash
soffice --headless --convert-to pdf pres_build/outputs/[file].pptx
pdftoppm -jpeg -r 110 pres_build/outputs/[file].pdf slide
```

Check every slide for:
- Text overflow or clipping
- Element overlap
- Colour contrast (especially text on coloured backgrounds)
- Accuracy vs content plan
- Tone consistency

Fix and rebuild if defects found.

---

## PHASE 6 — DELIVERY

1. Copy final `.pptx` to `pres_build/outputs/`
2. Present to user with summary of slides built
3. Flag any omitted data points or skipped slides
4. Note: "Works in Microsoft PowerPoint and Google Slides"

---

## HARD RULES

1. **NO ESTIMATION.** Every number must trace to a named source.
2. **No improvisation at build time.** All text from `content_plan.md`.
3. **Action titles only.** Full-sentence insights, never topic labels.
4. **Do not modify layout grid or spacing.** Only change content within compositions.
5. **Brand tokens only.** Never hardcode colours or fonts.
6. **Cover = dark bg + arc.** No photo, no overlay.
7. **Include/exclude slides based on data.** Never add empty slides.
8. **Center-align all panel content.** Stat boxes, callout bars, banners, source footnotes = centered. Left-align only for: table label columns, bullet/narrative paragraphs, bios, card body text.
9. **Icons are context-driven.** Pick from recoloured icon set based on slide content. If user wants to change icons, show all available options.
10. **Persist brand.** After first extraction, save to `pres_build/brands/` for reuse.
11. **Structure requires approval.** Never build without user confirming the slide structure.
12. **Skip photo compositions if no images provided.** Don't leave blank cards.

---

## ARC GENERATION REFERENCE

The decorative arcs are quarter-circle / wedge shapes. Generate at these sizes:

| Asset | Position | Size (inches) | Colour |
|-------|----------|---------------|--------|
| `cover_arc.png` | Right edge, full height | 4.2w × 5.6h | Primary |
| `divider_tl.png` | Top-left corner | 3.7w × 1.0h | Primary |
| `divider_br.png` | Bottom-right corner | 4.2w × 1.3h | Light |
| `disclaimer_bl.png` | Bottom-left | 1.7w × 1.7h | Light |
| `disclaimer_r1.png` | Right edge | 2.7w × 5.6h | Primary (30% opacity) |
| `disclaimer_r2.png` | Right edge (inner) | 2.5w × 5.6h | Primary |

Generate as transparent PNGs at 2x resolution (multiply inches by 192 for pixels).

---

## ICON GUIDE

All 109 icons from the base set. Pick based on context:

**Application & Status:** appDownload, appSend, appDeclined, appPaused, appPending, appReviewSearch, badge, calendar, checkCircle, checklist, createPen, declined, paused, pushpin, questionCircle, search, accountAdd, accountCircle, contactCard

**Financial & Document:** certification, finance, documentPound, documentEuro, documentList, documentUpload, documentAttach, documentChart, documentView, receipt, receiptEuro, receiptList, clipboardChart

**Charts & Data:** chart, chartGrow, lineChart, pieChart, monitorGrowth, monitorChart, presentationGrowth, presentationChart, data

**Security & Identity:** face_id, faceId1, fingerprint, padlock, securityLock, securityShield, cardLock, password, passcode, passwordDots, safe, stamp

**Business & Property:** briefcase, building, directions, factory, globe, openDoors, route, scales, smallBusinessBuilding, legalBriefcase, legal, calculator, wallet, websiteWWW, fast2, upload

**Industry-Specific:** industry8, industry10, construction, constructionCrane, crane, safety, truck, truck1, tractor, tractorFarming, foodAndDrink, hospitality, bakery, healthcare1, healthcare2, carehome, carDealer, mechanics, robotics, cycling, gym, golfGaming, iceSkating, beachUmbrella, sleigh, church, education, telescope, marketing, shopping, ecommerce, retail

**People & Growth:** handshake, team, person, grow, grow2, shootingStar, star1, hourglass

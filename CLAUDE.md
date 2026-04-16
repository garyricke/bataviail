# BataviaIL AI2026 — Project Context for Claude

## What This Is
A single self-contained `index.html` community portal for Batavia, IL.
Persona-based UX: user picks one of 7 personas → dashboard tailors content and org listings.

---

## Current State (as of 2026-03-02)

### File Sizes
- `index.html` — ~3.0MB (all assets embedded as base64)
- `generated_imgs/` — raw nano-banana PNGs (2K, ~2-4MB each)
- `personas/` — 400×400 JPEG persona portraits

### Dashboard Structure (Screen 2)
The old two-part header (`<header class="dash-header">` + `<div class="dash-hero">`) was **replaced** with a single ultrawide persona banner:

```html
<div class="persona-banner" id="persona-banner">
  <img class="persona-banner-bg" id="persona-banner-bg" src="" alt="">
  <div class="persona-banner-overlay">
    <button class="persona-banner-logo" onclick="showPersonaSelector()">
      <img class="batavia-logo" src="[base64 SVG]" alt="Batavia, IL">
      <span class="change-hint">↩ change persona</span>
    </button>
    <div class="persona-banner-info">
      <h2 id="dash-hero-title"></h2>
      <p id="dash-hero-desc"></p>
    </div>
  </div>
</div>
```

**Banner aspect ratios (CSS):**
- Default: `55/8` (ultrawide, ~6.875:1)
- ≤900px: `4/1`
- ≤600px: `3/1` (description text hidden)

**`object-position: center 18%`** — keeps heads in frame when the extreme-wide crop is applied.

### JS: PH Object
`const PH = { family, teen, socialite, settler, senior, business, volunteer }` — 7 base64 JPEG banner images (~3.1MB total). Defined immediately after `const PI = {...}` (the persona portrait images).

`showDashboard()` sets: `document.getElementById('persona-banner-bg').src = PH[p.id] || '';`

### Banner Images
7 photorealistic 16:9 panoramic collages, generated with `gemini-2.5-flash-image` at 2K, compressed to JPEG q78 at 1600px max:

| Key | Persona | Subject |
|---|---|---|
| `family` | Family Explorer | Hispanic mom + 2 kids |
| `teen` | Teen & Young Leader | Black teenage girl |
| `socialite` | Downtown Socialite | White woman late-20s, downtown |
| `settler` | New-to-Batavia Settler | Mixed-race couple with SOLD sign |
| `senior` | Senior & Care Circle | White woman 70s |
| `business` | Small Business Builder | South Asian man 40s |
| `volunteer` | Community Helper | Black woman with volunteer badge |

Source PNGs in `generated_imgs/` prefixed `banner2_*.jpg` (v2, the live ones).

---

## Key Architecture Facts

### Single HTML File
All assets are embedded — no external fetches at runtime:
- Base64 Batavia SVG logo (~43KB)
- `const PI = {...}` — 7 persona portrait base64 JPEGs (~495KB)
- `const PH = {...}` — 7 banner base64 JPEGs (~3.1MB)
- `const ORGS = [...]` — 293-org JS array (~283KB JSON)

### 3 Screens
1. `#screen-personas` — persona selector grid (default active)
2. `#screen-dashboard` — persona banner + events + orgs
3. Org modal — inline for standard orgs; external tab for pro orgs

### Pro Orgs (open external URL)
- Chuck's Cheeseburgers → https://chuckscheeseburgers.netlify.app
- Coffee & Sawdust → https://coffeeandsawdust.netlify.app
- INCubator@BHS → https://bhsincubator.org
- Water Street Studios → https://waterstreetstudios-publicart.org

### Brand Colors (CSS vars)
```css
--navy: #292F7B
--sky: #00ADEF
--green: #60C560
--gray: #7E7F81
```

### SVG Logo Rule
Always embed SVG logos as `<img src="data:image/svg+xml;base64,...">` — never inline `<svg>`. Inline SVGs inherit body `color` which corrupts white fills.

---

## 7 Personas

| ID | Name | Color | Audience |
|---|---|---|---|
| `family` | Family Explorer | #60C560 green | Parents + kids |
| `teen` | Teen & Young Leader | #00ADEF sky | Ages 13–22 |
| `socialite` | Downtown Socialite | #8b5cf6 purple | Ages 21–35 |
| `settler` | New-to-Batavia Settler | #f59e0b amber | New residents |
| `senior` | Senior & Care Circle | #06b6d4 cyan | Ages 65+ |
| `business` | Small Business Builder | #ef4444 red | Entrepreneurs |
| `volunteer` | Community Helper | #10b981 emerald | Volunteers |

---

## Data
- `batavia_businesses_with_summaries.json` — source JSON (not embedded in HTML)
- Structure: `{ metadata, bataviaLocal: [291 orgs], regional: [230 orgs] }`
- Membership levels: Standard (229), Gold (53), Platinum (9)
- Org fields: name, memberId, address, city, zip, phone, membershipLevel, description, logoUrl, categories, website, contacts[], socialMedia{}, summary, images[], hours

---

## Image Generation Notes
- Tool: nano-banana MCP (`gemini-2.5-flash-image`)
- Banner prompts: 16:9, 2K, photorealistic panoramic collage, subject centered bust portrait (waist-up, full head visible), activity scenes fill left and right thirds
- After generation: `sips -Z 1600 img.png -s format jpeg -s formatOptions 78 --out img.jpg`
- Base64 encode with Python: `base64.b64encode(open(f).read()).decode()`

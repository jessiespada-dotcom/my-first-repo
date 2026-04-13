---
description: Generate WHOOP-branded reveal.js HTML slide decks from any content source. Creates multiple design variations, supports iterative refinement, and exports to PPTX or PDF.
user-invocable: true
---

# WHOOP Presentation Generator

You are a WHOOP presentation specialist. Follow the steps below. Read `DESIGN_GUIDE.md` before generating any HTML — it is the authoritative brand reference.

## Arguments

$ARGUMENTS

If a URL or content source is passed as an argument, use it directly and skip asking for it.

---

## Step 1 — Preflight

### Atlassian MCP (required if fetching from an Atlassian URL)

Check that the WHOOP Atlassian MCP server is configured. If not:
```
⚠️  WHOOP Atlassian MCP not detected.
Configure it here:
https://whoopinc.atlassian.net/wiki/spaces/SW/pages/4850188600/Atlassian+MCP+Server
Then restart Claude Code and re-run /whoop-presentation.
```
**STOP** if the user's source is an Atlassian URL and the MCP is unavailable.

### Dependencies (run silently, fix if missing)

```bash
node -v                    # needed for PDF export via decktape
python3 -c "import pptx, PIL, bs4" 2>/dev/null \
  || python3 -m pip install --quiet python-pptx pillow beautifulsoup4
ls ~/Library/Fonts/ /Library/Fonts/ 2>/dev/null | grep -i proxima || true
```

- If Node missing: install via `brew install node` (macOS) or `apt-get install nodejs` (Linux)
- If Python packages missing: auto-install (shown above)
- If Proxima Nova not found: print `ℹ️  Using Helvetica Neue fallback` and continue

### Brand assets

Copy assets from the plugin's `resources/brand/` directory (bundled with this plugin):
```bash
BRAND="${CLAUDE_SKILL_DIR}/resources/brand"
cp "$BRAND/logos/wordmark_white.png" ./whoop_logo_white.png 2>/dev/null || true
cp "$BRAND/logos/wordmark_black.png" ./whoop_logo_black.png 2>/dev/null || true
cp "$BRAND/logos/puck_white.png"     ./whoop_puck_white.png 2>/dev/null || true
cp "$BRAND/logos/puck_black.png"     ./whoop_puck_black.png 2>/dev/null || true
cp "$BRAND/images/"active_*.jpg  ./ 2>/dev/null || true
cp "$BRAND/images/"casual_*.jpg  ./ 2>/dev/null || true
cp "$BRAND/images/"band_*.jpg    ./ 2>/dev/null || true
cp "$BRAND/icons/"icon_*.png     ./ 2>/dev/null || true
```

**Logo usage**: `whoop_logo_white.png` on dark backgrounds, `whoop_logo_black.png` on light. No `filter: invert()` needed — use the correct variant directly.

**Image usage** (all 1280×720 JPG, use full-bleed with scrim):
- Dark/action theme → `active_NN.jpg`
- Light/casual theme → `casual_NN.jpg`
- Product/device → `band_NN.jpg`

**Do NOT reference** `cover_*.png` or `template_cover_*.png` — deprecated and removed.

If assets are missing, fall back to CSS gradients and the inline SVG wordmark in DESIGN_GUIDE.md.

---

## Step 2 — Gather Inputs

```
Let's build your WHOOP presentation:

1. Content source? (Atlassian URL, GitHub link, pasted text, or a brief outline)
2. Which template?
     a) Dark brand  — dark-brand-template.html    (marketing, all-hands, exec)
     b) Light eng   — light-engineering-template.html  (engineering, internal, technical)
     c) Original    — template.html               (fully custom WHOOP dark, built from scratch)
   [default: a]
3. How many design variations? [default: 3]
4. Any specific vibe? (e.g. "darker", "data-forward", "minimal") [default: standard WHOOP dark]
```

Wait for answers. If an Atlassian URL is provided, use the WHOOP Atlassian MCP. For GitHub URLs, use the `gh` CLI or WebFetch. For pasted text or outlines, use the content directly.

---

## Step 3 — Fetch & Outline

Fetch the content from the provided source, then output a proposed slide structure for user approval:

```
📋 Proposed structure:
Slide 1:  Title — [title]
Slide 2:  Agenda (if 4+ sections)
Slide 3:  [Section divider]
...
Slide N:  Summary / Next Steps

Does this work, or would you like to adjust?
```

Wait for confirmation before generating.

---

## Step 4 — Generate N HTML Variations

Write `slides-v1.html` through `slides-vN.html` in the current directory.

**Start from the template chosen in Step 2** — do not build CSS or boilerplate from scratch:

| Choice | File | When to use |
|--------|------|-------------|
| **a** Dark brand | `dark-brand-template.html` | Marketing, all-hands, exec, external — full dark WHOOP brand with cover photo |
| **b** Light eng  | `light-engineering-template.html` | Engineering RFCs, internal reviews, technical deep-dives — white background, right-side WHOOP puck |
| **c** Original   | `template.html` | Fully custom dark deck built from scratch using WHOOP CSS primitives |

Copy the chosen file as the base for every variation. Replace all placeholder text (`Presentation Title`, `Slide Title`, `Author Name`, `bullet point goes here`, etc.) with real content. Add, remove, or reorder `<section>` blocks as needed.

**Architecture:** 1280×720, Proxima Nova (Helvetica Neue fallback), WHOOP color variables. Dark brand and Original templates use a dark gradient; Light eng uses white with grey chrome.

**Vary across versions** (content is identical across all — only layout differs):
- Cover: centered vs. left-aligned title
- Dividers: minimal label vs. large typographic treatment
- Body: 2-col vs. 3-col grids where content supports it
- Logo: header-only vs. header + title watermark + corner marks on content slides
- Background: pure CSS gradient vs. a `casual_*` or `active_*` jpg (opacity 0.12–0.18 scrim) on title/dividers

**Always include** the WHOOP wordmark in: slide header, title slide (large), section dividers. Use `wordmark_white.png` on dark backgrounds, `wordmark_black.png` on light — no `filter: invert()` needed.

**Photography**: match background image category to slide purpose — `band_*` for product/device slides, `active_*` for performance/data slides, `casual_*` for culture/lifestyle/divider slides. Pick varied images across slides.

**Terminology**: use correct WHOOP terms from DESIGN_GUIDE.md — capitalize Sleep, Strain, Recovery, Stress as metrics; use WHOOP One/Peak/Life for memberships; say Member not user.

Do **not** produce `deck.json` or `final-slides.html` at this stage.

---

## Step 5 — Version Selection Gate (STOP)

After writing the HTML files, **stop**. Do not export, do not copy to `final-slides.html`.

```
✅  Generated [N] variations in [directory]
   • slides-v1.html
   • slides-v2.html  (open with arrow keys to navigate)
   ...

What next?
  A) Refine a version — e.g. "Refine v2"
  B) More variations — e.g. "Generate 2 more, lighter feel"
  C) Finalize — e.g. "Finalize v2" (triggers export)
```

---

## Step 6 — Refinement Loop

Edit only the chosen `slides-vN.html`. Use targeted in-place edits (Edit tool), not full rewrites.

After each batch of changes, print a short changelog:
```
[v2 — updated]
• Slide 3: title → "Q1 PERFORMANCE"
• Slide 5: added callout box
Refresh browser to preview.
```

Trigger export when user says: `"finalize"`, `"export"`, `"ship it"`, or `"this looks good"`.

---

## Step 7 — Export

```bash
cp slides-vN.html final-slides.html

# PPTX
python3 scripts/export_pptx.py --html final-slides.html --out final-slides.pptx

# PDF (fallback)
npx decktape reveal final-slides.html final-slides.pdf --size 1280x720
```

Print paths of produced files on completion.

---

## Step 8 — Share (optional)

After export, ask the user:

```
🌐  Would you like to upload this to the WHOOP Presentation Sharer so you can share a link?
    It will be accessible to any WHOOP employee on VPN or in the office with their Google account.

    (yes / no)
```

If the user says yes, suggest a descriptive key derived from the presentation title — slugified, lowercase, hyphens, `.html` suffix (e.g. `q2-engineering-review.html`). Then ask:

```
📁  Where should it be stored?
    Suggested: <descriptive-name>.html

    You can organize it into a folder — e.g. "engineering/<descriptive-name>.html".
    Confirm or enter a different path.
```

**Never accept `final-slides.html` or `slides-vN.html` as the destination name** — always derive a descriptive name from the presentation content. Once the user confirms the path, run:

```bash
bash "${CLAUDE_SKILL_DIR}/scripts/upload_presentation.sh" final-slides.html [confirmed-key]
```

**Handle exit codes:**

| Code | Meaning | What to tell the user |
|------|---------|----------------------|
| `0`  | Success — print the `🔗` URL from stdout | — |
| `2`  | No file argument | Report internal error — file path was not passed to the script |
| `3`  | File not found | "The file `final-slides.html` wasn't found — make sure export completed successfully." |
| `4`  | `WHOOP_API_KEY_DEV` not set | "Your `WHOOP_API_KEY_DEV` environment variable isn't set. Add it to your shell profile and restart Claude Code." |
| `5`  | HTTP error from the server | "The upload request failed. Check that you're on VPN or in the office and try again." |

---

## Key Rules

- **Phase gate**: no export before user says "finalize"
- **Refinement scope**: edit only the chosen version file
- **Colors**: only official WHOOP palette (see DESIGN_GUIDE.md) — no off-brand colors
- **Typography**: Proxima Nova Bold uppercase for headlines; Helvetica Neue fallback
- **Canvas**: 1280×720, margin: 0
- **Content**: bullets and short phrases only — no dense paragraphs
- **All variations**: identical content, different layout treatment

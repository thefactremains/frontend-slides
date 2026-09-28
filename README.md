# Frontend Slides

A coding-agent skill for creating HTML presentations with lots of animation, either from scratch or by converting PowerPoint files. It ships as a Claude Code plugin and as a standalone skill. Other coding agents that can read files and run shell commands can use the core `SKILL.md` too.

> **This is a maintained fork** of [zarazhangrui/frontend-slides](https://github.com/zarazhangrui/frontend-slides). It fixes a privacy leak in the deploy script, broken PDF and print export, and gaps in PowerPoint extraction, and it makes the skill instructions consistent with each other. See [Changes in this fork](#changes-in-this-fork). The design system, presets and templates are the upstream author's work.

## 📺 Walkthrough & Tutorial

New here? This beginner-friendly video from the original author walks you through the whole thing, start to finish.

<a href="https://www.youtube.com/watch?v=372Iksaz8b0" title="Frontend Slides — walkthrough & tutorial (beginner-friendly)">
  <img src="https://img.youtube.com/vi/372Iksaz8b0/maxresdefault.jpg" alt="Watch the Frontend Slides walkthrough and tutorial on YouTube" width="100%" />
</a>

> ▶️ **[Watch on YouTube →](https://www.youtube.com/watch?v=372Iksaz8b0)**

## What This Does

**Frontend Slides** helps non-designers make good-looking web presentations without knowing CSS or JavaScript. It works by "show, don't tell": instead of asking you to describe your taste in words, it generates three visual previews and you pick the one you like.

Here is a deck about the skill, made with the skill:

https://github.com/user-attachments/assets/ef57333e-f879-432a-afb9-180388982478

### Key Features

- **Zero dependencies.** Each deck is one HTML file with inline CSS and JS. No npm, build tools or frameworks.
- **Visual style discovery.** You react to three rendered title-slide previews instead of answering abstract design questions.
- **Fixed 16:9 stage.** Every slide is laid out at 1920×1080 and scaled as a whole, so it looks the same on a projector, a laptop and a phone (letterboxed, never reflowed).
- **Two density modes.** *Speaker-led* decks get fewer words and bigger type. *Reading-first* decks get self-contained slides for async review.
- **PowerPoint conversion.** Extracts slide text, grouped shapes, tables, embedded images, picture placeholders and speaker notes.
- **12 safe presets and 34 bold templates.** Curated styles that avoid generic "AI slop" looks. The bold templates load progressively, so only the one you pick gets read in full.
- **Inline editing.** Press `E` (or hover the top-left corner) to edit text in the browser, then save with Ctrl/Cmd+S.
- **Sharing.** Deploy to a public Vercel URL or export a PDF with one command.

## Installation

### Claude Code: plugin (recommended)

Run these as **two separate messages** in Claude Code:

```text
/plugin marketplace add https://github.com/thefactremains/frontend-slides
```

```text
/plugin install frontend-slides@frontend-slides
```

Use the HTTPS URL. The short `owner/repo` form can make Claude Code try SSH, which fails if GitHub isn't in your `known_hosts`.

Invoke it with `/frontend-slides:frontend-slides`. Plugin skills are namespaced as `/plugin-name:skill-name`.

If you already added the upstream marketplace under the same name, remove it first with `/plugin marketplace remove frontend-slides`.

### Claude Code: standalone skill

This works in the Claude Code CLI and the Code tab of the Claude desktop app.

```bash
git clone https://github.com/thefactremains/frontend-slides.git /tmp/frontend-slides
```

```bash
mkdir -p ~/.claude/skills && cp -R /tmp/frontend-slides/plugins/frontend-slides/skills/frontend-slides ~/.claude/skills/
```

Start a new session and invoke it with `/frontend-slides`. Standalone skills are not namespaced. To update later, pull the repo and run the copy again.

### Claude app and Cowork (skill upload)

Claude.ai, the Claude desktop app and Cowork use the skills on your Claude account, so the skill has to be uploaded as a zip:

```bash
cd /tmp/frontend-slides/plugins/frontend-slides/skills && zip -qr ~/Desktop/frontend-slides-skill.zip frontend-slides -x '*.DS_Store'
```

In the Claude app, open the Skills settings, choose to upload a skill, and select `frontend-slides-skill.zip`. Code execution must be enabled for the scripts (PowerPoint extraction, PDF export, deploy) to run.

### Other coding agents

Codex, Gemini CLI, OpenCode, Kimi Code and other local agents can use the same skill. Point the agent at this repo and ask it to use the Frontend Slides skill:

```text
https://github.com/thefactremains/frontend-slides
```

The agent should start from `SKILL.md` and load only the support files it needs (`STYLE_PRESETS.md`, `viewport-base.css`, `html-template.md`, `animation-patterns.md`, `bold-template-pack/`, `scripts/`). Scripts must be run from the skill's own folder path, not the project directory.

## Usage

### Create a new presentation

```text
/frontend-slides:frontend-slides

> "I want to create a pitch deck for my AI startup"
```

(Use `/frontend-slides` for a standalone install. In other agents, just ask for the Frontend Slides skill.)

The skill will:

1. Ask four questions at once: **purpose**, **length**, whether your **content** is ready, and **density** (speaker-led or reading-first).
2. If you share images, check each one and plan the outline around the usable ones.
3. Generate **3 title-slide previews**: one safe preset, at least one bold template, and one wildcard (another template or a custom design). If you named a vibe or style, it uses that.
4. Build the full deck in the style you pick, then check it for overflowing text and overlapping panels.
5. Open it in your browser and offer to deploy or export it.

### Convert a PowerPoint

```text
/frontend-slides:frontend-slides

> "Convert my presentation.pptx to a web slideshow"
```

The skill runs `scripts/extract-pptx.py`. It extracts titles, body text (including text in grouped shapes), tables as rows, embedded images (including picture placeholders) and speaker notes, then confirms the extracted content with you before styling it.

**Not extracted:** charts, SmartArt, and images that are linked rather than embedded. Those need to be recreated or supplied separately.

### Enhance an existing deck

Point the skill at an existing HTML presentation and describe the change. It keeps the fixed 16:9 stage and the deck's density mode, and it splits slides instead of cramming content in.

### Navigating and editing a deck

| Action | How |
| --- | --- |
| Next / previous slide | Arrow keys, Space, Page Up/Down, mouse wheel, swipe |
| Edit text | Press `E` or hover the top-left corner, then click any text |
| Save edits | Ctrl/Cmd+S (edits also auto-save to the browser's localStorage) |
| Restyle | Change the `:root` CSS variables (colors, fonts, sizes) at the top of the file |
| Print | Browser print gives one 16:9 page per slide, with every animated element shown in its final state |

## Sharing Your Presentations

Script paths below are relative to the skill folder, for example `~/.claude/skills/frontend-slides/scripts/`. The skill resolves this path itself when it runs them for you.

### Deploy to a live URL (Vercel)

```bash
bash scripts/deploy.sh ./presentation.html
```

```bash
bash scripts/deploy.sh ./my-deck/
```

- **Single HTML file:** uploads the HTML (as `index.html`), the local files it references through `src`, `href` or `url(...)`, and an `assets/` folder next to it if there is one. Nothing else in that folder is uploaded. References that climb out of the folder (`../`) are skipped.
- **Folder:** uploads **everything in the folder** as-is. It must contain an `index.html`, and it shouldn't contain anything you don't want public.
- The URL is public. Redeploying updates the same URL. To take it down, delete the project at https://vercel.com/dashboard.
- Needs Node.js and a free Vercel account. Run `vercel login` yourself in a terminal before the first deploy. The login is interactive and can't finish inside an agent's non-interactive shell.

### Export to PDF

```bash
bash scripts/export-pdf.sh ./presentation.html
```

```bash
bash scripts/export-pdf.sh ./presentation.html ./slides.pdf --compact
```

- Screenshots each slide at 1920×1080 (or 1280×720 with `--compact` for a smaller file) and combines them into one PDF.
- Animations are frozen in their final state, so every slide exports complete.
- Installs Playwright into a temp folder on each run. If the Chromium download fails (offline, firewall, CDN outage), it falls back to your installed Google Chrome.
- Local images must use relative paths, and slides must use `class="slide"`.

## Requirements

- A local agent with filesystem access and shell commands (Claude Code, Cowork, or similar)
- **PowerPoint conversion:** Python 3 with `python-pptx` (`pip install python-pptx`)
- **Image processing** (circular crops, resizing oversized images): `Pillow` (`pip install Pillow`)
- **Deploy:** Node.js and a free Vercel account
- **PDF export:** Node.js. Playwright installs automatically, and Google Chrome is used as a fallback.

## Troubleshooting

| Problem | Fix |
| --- | --- |
| PDF export: "Failed to install Chromium" | Install Google Chrome (the script falls back to it), or run `npx playwright install chromium` on a network that can reach Playwright's CDN |
| PDF export: "0 slides found" | The deck doesn't use `class="slide"`. Decks from this skill always do; external decks may not. |
| Images missing in PDF or on the deployed site | Use relative paths (`assets/logo.png`), not absolute ones (`/Users/you/...`). For decks with many assets, put everything in one folder and deploy the folder. |
| Deploy stuck at login | Run `vercel login` in your own terminal, then retry |
| `/frontend-slides` not found | A plugin install uses `/frontend-slides:frontend-slides`. A new skill only shows up in a new session. |
| Slides look too small on a phone | Expected. The stage keeps 16:9 and letterboxes instead of reflowing. Rotate to landscape. |

## Included Styles

### Safe presets (`STYLE_PRESETS.md`)

| Dark | Light | Specialty |
| --- | --- | --- |
| **Bold Signal**: vibrant card on dark | **Notebook Tabs**: paper with colorful tabs | **Neon Cyber**: particles, neon glow |
| **Electric Studio**: clean split panel | **Pastel Geometry**: vertical pills | **Terminal Green**: hacker aesthetic |
| **Creative Voltage**: electric blue + neon | **Split Pastel**: two-color split | **Swiss Modern**: Bauhaus grid |
| **Dark Botanical**: elegant, warm accents | **Vintage Editorial**: witty, geometric | **Paper & Ink**: drop caps, pull quotes |

### Bold template pack (`bold-template-pack/`)

34 design systems from [`beautiful-html-templates`](https://github.com/zarazhangrui/beautiful-html-templates), such as **Neo-Grid Bold**, **Editorial Tri-Tone**, **Creative Mode**, **Broadside**, **Signal** and **Vellum**.

The agent reads the compact `selection-index.json` first. For previews it loads only the shortlisted templates' small `preview.md` cards, and it loads the full `design.md` for exactly one template, after you pick it. The source templates' `template.html` files are not bundled.

<details>
<summary><strong>Bold template gallery (34 templates, 3 screenshots each)</strong></summary>

### [Soft Editorial](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/soft-editorial/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/soft-editorial-4.png" width="32.5%" alt="Soft Editorial — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/soft-editorial-6.png" width="32.5%" alt="Soft Editorial — slide 6" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/soft-editorial-10.png" width="32.5%" alt="Soft Editorial — slide 10" />
</p>

> Cormorant Garamond serif on warm paper with sage, blush, and lemon accents.

### [Editorial Forest](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/editorial-forest/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/editorial-forest-1.png" width="32.5%" alt="Editorial Forest — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/editorial-forest-2.png" width="32.5%" alt="Editorial Forest — slide 2" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/editorial-forest-5.png" width="32.5%" alt="Editorial Forest — slide 5" />
</p>

> Forest green, dusty pink, and warm cream in Source Serif 4 — quiet, intentional quarterly-review aesthetic.

### [Pin & Paper](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/pin-and-paper/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/pin-and-paper-1.png" width="32.5%" alt="Pin & Paper — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/pin-and-paper-11.png" width="32.5%" alt="Pin & Paper — slide 11" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/pin-and-paper-3.png" width="32.5%" alt="Pin & Paper — slide 3" />
</p>

> Yellow paper with safety-pin illustrations, ink-blue handwritten Caveat, paper-grain texture.

### [Sakura Chroma](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/sakura-chroma/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/sakura-chroma-1.png" width="32.5%" alt="Sakura Chroma — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/sakura-chroma-3.png" width="32.5%" alt="Sakura Chroma — slide 3" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/sakura-chroma-4.png" width="32.5%" alt="Sakura Chroma — slide 4" />
</p>

> Vintage Japanese cassette-package aesthetic: cream paper, diagonal rainbow ribbons, condensed bold type, JIS-style spec checkboxes.

### [Stencil & Tablet](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/stencil-tablet/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/stencil-tablet-1.png" width="32.5%" alt="Stencil & Tablet — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/stencil-tablet-3.png" width="32.5%" alt="Stencil & Tablet — slide 3" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/stencil-tablet-8.png" width="32.5%" alt="Stencil & Tablet — slide 8" />
</p>

> Bone paper with stencil-cut headlines and a six-color earth palette: archaeology meets brand.

### [Cobalt Grid](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/cobalt-grid/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/cobalt-grid-1.png" width="32.5%" alt="Cobalt Grid — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/cobalt-grid-3.png" width="32.5%" alt="Cobalt Grid — slide 3" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/cobalt-grid-5.png" width="32.5%" alt="Cobalt Grid — slide 5" />
</p>

> Electric cobalt italic serifs on a graph-paper canvas, anchored by stair-stepped pixel-glitch decorations and slim hairline rules.

### [Vellum](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/vellum/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/vellum-1.png" width="32.5%" alt="Vellum — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/vellum-4.png" width="32.5%" alt="Vellum — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/vellum-8.png" width="32.5%" alt="Vellum — slide 8" />
</p>

> Deep navy canvas with warm-yellow italic Cormorant serifs and a single dusty teal accent. A quiet, scholarly aesthetic.

### [Emerald Editorial](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/emerald-editorial/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/emerald-editorial-1.png" width="32.5%" alt="Emerald Editorial — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/emerald-editorial-3.png" width="32.5%" alt="Emerald Editorial — slide 3" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/emerald-editorial-6.png" width="32.5%" alt="Emerald Editorial — slide 6" />
</p>

> Magazine-cover business deck: emerald + navy + paper with double-rule masthead ornaments and a heavy Bodoni-style display serif.

### [Neo-Grid Bold](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/neo-grid-bold/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/neo-grid-bold-1.png" width="32.5%" alt="Neo-Grid Bold — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/neo-grid-bold-3.png" width="32.5%" alt="Neo-Grid Bold — slide 3" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/neo-grid-bold-8.png" width="32.5%" alt="Neo-Grid Bold — slide 8" />
</p>

> Editorial neo-brutalism with a single neon yellow accent on off-white paper.

### [Editorial Tri-Tone](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/editorial-tri-tone/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/editorial-tri-tone-1.png" width="32.5%" alt="Editorial Tri-Tone — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/editorial-tri-tone-4.png" width="32.5%" alt="Editorial Tri-Tone — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/editorial-tri-tone-3.png" width="32.5%" alt="Editorial Tri-Tone — slide 3" />
</p>

> Three-color editorial system: dusty pink, mustard cream, and deep burgundy, set in Bricolage + Instrument Serif.

### [Creative Mode](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/creative-mode/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/creative-mode-1.png" width="32.5%" alt="Creative Mode — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/creative-mode-4.png" width="32.5%" alt="Creative Mode — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/creative-mode-6.png" width="32.5%" alt="Creative Mode — slide 6" />
</p>

> Cream paper canvas with confident multi-color (green, pink, orange, yellow) accents and Archivo Black display.

### [Monochrome](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/monochrome/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/monochrome-1.png" width="32.5%" alt="Monochrome — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/monochrome-4.png" width="32.5%" alt="Monochrome — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/monochrome-12.png" width="32.5%" alt="Monochrome — slide 12" />
</p>

> Ivory ledger paper with all-black type; Lora serif headlines, Jost body, no color at all.

### [People's Platform (Block & Bold)](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/peoples-platform/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/peoples-platform-1.png" width="32.5%" alt="People's Platform (Block & Bold) — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/peoples-platform-4.png" width="32.5%" alt="People's Platform (Block & Bold) — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/peoples-platform-8.png" width="32.5%" alt="People's Platform (Block & Bold) — slide 8" />
</p>

> Activist poster energy: blue, orange, red on cream, with Alfa Slab + Caveat Brush.

### [Pink Script — After Hours](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/pink-script/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/pink-script-1.png" width="32.5%" alt="Pink Script — After Hours — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/pink-script-4.png" width="32.5%" alt="Pink Script — After Hours — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/pink-script-8.png" width="32.5%" alt="Pink Script — After Hours — slide 8" />
</p>

> Black canvas, hot pink accent, pearl-cream paper, Instrument Serif headlines: late-night editorial luxury.

### [8-Bit Orbit](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/8-bit-orbit/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/8-bit-orbit-1.png" width="32.5%" alt="8-Bit Orbit — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/8-bit-orbit-6.png" width="32.5%" alt="8-Bit Orbit — slide 6" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/8-bit-orbit-5.png" width="32.5%" alt="8-Bit Orbit — slide 5" />
</p>

> Pixel-art neon arcade aesthetic on a deep navy void.

### [BlockFrame](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/block-frame/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/block-frame-1.png" width="32.5%" alt="BlockFrame — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/block-frame-4.png" width="32.5%" alt="BlockFrame — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/block-frame-8.png" width="32.5%" alt="BlockFrame — slide 8" />
</p>

> Neobrutalist deck with pastel-neon color blocks and chunky black borders.

### [Blue Professional](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/blue-professional/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/blue-professional-1.png" width="32.5%" alt="Blue Professional — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/blue-professional-6.png" width="32.5%" alt="Blue Professional — slide 6" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/blue-professional-8.png" width="32.5%" alt="Blue Professional — slide 8" />
</p>

> Cream paper background with electric cobalt blue accents; clean modern professional.

### [Bold Poster](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/bold-poster/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/bold-poster-1.png" width="32.5%" alt="Bold Poster — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/bold-poster-4.png" width="32.5%" alt="Bold Poster — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/bold-poster-8.png" width="32.5%" alt="Bold Poster — slide 8" />
</p>

> Editorial poster aesthetic with massive Shrikhand display and a single fire-engine red accent.

### [Broadside](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/broadside/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/broadside-1.png" width="32.5%" alt="Broadside — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/broadside-4.png" width="32.5%" alt="Broadside — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/broadside-13.png" width="32.5%" alt="Broadside — slide 13" />
</p>

> Dark editorial canvas with a single fire orange accent and bilingual Latin/Chinese type stack.

### [Capsule](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/capsule/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/capsule-1.png" width="32.5%" alt="Capsule — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/capsule-4.png" width="32.5%" alt="Capsule — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/capsule-8.png" width="32.5%" alt="Capsule — slide 8" />
</p>

> Modular pill-shaped cards on warm bone with a full pastel-pop palette.

### [Cartesian](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/cartesian/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/cartesian-1.png" width="32.5%" alt="Cartesian — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/cartesian-4.png" width="32.5%" alt="Cartesian — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/cartesian-8.png" width="32.5%" alt="Cartesian — slide 8" />
</p>

> Quiet warm-neutral palette with classical Playfair serifs; tasteful and unhurried.

### [Coral](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/coral/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/coral-1.png" width="32.5%" alt="Coral — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/coral-4.png" width="32.5%" alt="Coral — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/coral-8.png" width="32.5%" alt="Coral — slide 8" />
</p>

> Cream and coral on near-black, set in oversized Bebas Neue.

### [Daisy Days](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/daisy-days/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/daisy-days-1.png" width="32.5%" alt="Daisy Days — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/daisy-days-4.png" width="32.5%" alt="Daisy Days — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/daisy-days-8.png" width="32.5%" alt="Daisy Days — slide 8" />
</p>

> Cheerful pastel deck with hand-drawn daisies, stars, and rainbows. Friendly, soft, and warm.

### [Grove](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/grove/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/grove-1.png" width="32.5%" alt="Grove — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/grove-4.png" width="32.5%" alt="Grove — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/grove-8.png" width="32.5%" alt="Grove — slide 8" />
</p>

> Forest-green canvas with cream type, classical Playfair serifs, and a single rust accent.

### [Mat](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/mat/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/mat-1.png" width="32.5%" alt="Mat — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/mat-4.png" width="32.5%" alt="Mat — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/mat-8.png" width="32.5%" alt="Mat — slide 8" />
</p>

> Dark sage canvas with bone paper and burnt-orange accent; mid-century modern with wood undertones.

### [Playful](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/playful/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/playful-1.png" width="32.5%" alt="Playful — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/playful-6.png" width="32.5%" alt="Playful — slide 6" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/playful-8.png" width="32.5%" alt="Playful — slide 8" />
</p>

> Sun-warm peach background with Syne display: a friendly indie launch deck.

### [Raw Grid](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/raw-grid/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/raw-grid-1.png" width="32.5%" alt="Raw Grid — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/raw-grid-4.png" width="32.5%" alt="Raw Grid — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/raw-grid-8.png" width="32.5%" alt="Raw Grid — slide 8" />
</p>

> Neo-brutalist deck with thick borders, offset shadows, and a pink/sage/ink palette.

### [Retro Windows](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/retro-windows/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/retro-windows-1.png" width="32.5%" alt="Retro Windows — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/retro-windows-4.png" width="32.5%" alt="Retro Windows — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/retro-windows-8.png" width="32.5%" alt="Retro Windows — slide 8" />
</p>

> Windows 95 chrome: gray title bars, MS Sans Serif, pixel typography, full nostalgia.

### [Retro Zine](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/retro-zine/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/retro-zine-1.png" width="32.5%" alt="Retro Zine — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/retro-zine-4.png" width="32.5%" alt="Retro Zine — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/retro-zine-8.png" width="32.5%" alt="Retro Zine — slide 8" />
</p>

> Beige paper with green accent and Bebas Neue + Caveat: a riso-printed zine in HTML form.

### [Scatterbrain](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/scatterbrain/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/scatterbrain-1.png" width="32.5%" alt="Scatterbrain — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/scatterbrain-4.png" width="32.5%" alt="Scatterbrain — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/scatterbrain-8.png" width="32.5%" alt="Scatterbrain — slide 8" />
</p>

> Post-it inspired: pastel sticky notes, Caveat handwriting, Shrikhand and Zilla Slab type stack.

### [Signal](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/signal/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/signal-1.png" width="32.5%" alt="Signal — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/signal-18.png" width="32.5%" alt="Signal — slide 18" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/signal-8.png" width="32.5%" alt="Signal — slide 8" />
</p>

> Deep navy canvas with bone paper and a single muted-gold accent; institutional with quiet weight.

### [Studio](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/studio/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/studio-1.png" width="32.5%" alt="Studio — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/studio-4.png" width="32.5%" alt="Studio — slide 4" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/studio-8.png" width="32.5%" alt="Studio — slide 8" />
</p>

> Black canvas with electric-yellow type; high-voltage design studio aesthetic.

### [Biennale Yellow](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/biennale-yellow/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/biennale-yellow-1.png" width="32.5%" alt="Biennale Yellow — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/biennale-yellow-5.png" width="32.5%" alt="Biennale Yellow — slide 5" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/biennale-yellow-8.png" width="32.5%" alt="Biennale Yellow — slide 8" />
</p>

> Solar yellow on warm parchment with deep indigo serif and atmospheric sun-glow gradients. Dutch-editorial poster energy.

### [Long Table](https://github.com/zarazhangrui/beautiful-html-templates/tree/main/templates/long-table/)

<p>
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/long-table-1.png" width="32.5%" alt="Long Table — slide 1" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/long-table-3.png" width="32.5%" alt="Long Table — slide 3" />
  <img src="https://raw.githubusercontent.com/zarazhangrui/beautiful-html-templates/main/screenshots/long-table-7.png" width="32.5%" alt="Long Table — slide 7" />
</p>

> Warm cream and rust-red supper-club aesthetic with bold uppercase grotesk headlines, italic Fraunces, and pill-shaped outlined buttons.

</details>

## Architecture

The skill uses **progressive disclosure**: `SKILL.md` is a workflow map, and the supporting files are loaded only when a phase needs them.

| File | Purpose | Loaded when |
| --- | --- | --- |
| `SKILL.md` | Core workflow and rules | Always (skill invocation) |
| `STYLE_PRESETS.md` | 12 curated visual presets | Phase 2 (style selection) |
| `bold-template-pack/selection-index.json` | Compact metadata for shortlisting bold templates | Phase 2 (style selection) |
| `bold-template-pack/templates/*/preview.md` | Small style cards for shortlisted previews | Phase 2, after shortlisting |
| `bold-template-pack/templates/*/design.md` | Full design system for the chosen template | Phase 3, after you pick |
| `viewport-base.css` | Mandatory fixed-stage and print CSS | Phase 3 (generation) |
| `html-template.md` | HTML structure, controller, inline editing, images | Phase 3 (generation) |
| `animation-patterns.md` | Animation snippets and effect-to-feeling guide | Phase 3 (generation) |
| `scripts/extract-pptx.py` | PowerPoint content extraction | Phase 4 (conversion) |
| `scripts/deploy.sh` | Deploy to Vercel | Phase 6 (sharing) |
| `scripts/export-pdf.sh` | Export slides to PDF | Phase 6 (sharing) |

### Repository layout

The skill exists in two identical copies:

- **Repo root:** for agents that read the repo directly and for `git clone` installs
- **`plugins/frontend-slides/skills/frontend-slides/`:** what the Claude Code plugin installs and what the standalone and zip installs above copy

When you change the skill, change both copies and confirm they match:

```bash
for f in SKILL.md STYLE_PRESETS.md html-template.md viewport-base.css animation-patterns.md scripts bold-template-pack; do diff -rq "$f" "plugins/frontend-slides/skills/frontend-slides/$f"; done
```

Bump `version` in `plugins/frontend-slides/.claude-plugin/plugin.json` so plugin users get the update.

## Changes in this fork

Every fix below was reproduced first, against a test deck or `.pptx`, and re-tested after the change.

**Scripts**

- **`deploy.sh` privacy leak.** The file-reference regex matched only `src=` and `href=`, so every reference came out empty and resolved to the HTML's parent folder. Deploying a single HTML file uploaded *every file next to it* to a public URL. It now copies only the referenced files and skips URLs, `data:` URIs and `../` paths.
- **`deploy.sh`:** `assets/` no longer gets nested as `assets/assets/`. Filenames made only of non-ASCII characters (e.g. `演示.html`) no longer crash, and the temp directory can't collide with an existing folder.
- **`export-pdf.sh`:** slides after the first exported half-faded or blank, because screenshots were taken mid-transition and `.visible` was never set. Motion is now frozen, both `.active` and `.visible` are set, the deck's own controller is used, and every `reveal-*` class is forced to its final state.
- **`export-pdf.sh`:** no longer crashes on macOS's default bash 3.2 when run with no arguments or only `--compact`.
- **`export-pdf.sh`:** a relative output path (`slides.pdf`) used to be written inside the temp folder and deleted, while the script still reported success. The path is now resolved to an absolute one first.
- **`export-pdf.sh`:** falls back to an installed Google Chrome when the Chromium download fails.
- **`extract-pptx.py`:** now extracts text and images inside grouped shapes, tables, and pictures in picture placeholders, all previously dropped. It no longer emits empty text entries and writes UTF-8 JSON.

**Skill instructions and CSS**

- **`viewport-base.css`:** browser print used Letter-size portrait pages and printed later slides with their animated content missing. It now uses 1920×1080 pages with reveal elements in their final state.
- **`SKILL.md`:** scripts are now called by the skill-folder path (they were called relative to the user's project). Removed a reference to a "No images" answer that no question asks. Aligned the enhance-mode bullet limit with the density modes. Dropped the pointer to the unbundled `template.html`. Spelled out exactly what a deploy uploads.
- **`html-template.md`:** exposes `window.presentation` for the exporter and requires Ctrl/Cmd+S saving (the delivery message promised it). Navigation and the `E` key are ignored while editing text, and viewport units are removed from inside the fixed stage.
- **`animation-patterns.md`:** `reveal-scale`, `reveal-left` and `reveal-blur` had no end state, so anything using them stayed invisible. Removed leftover scroll-snap and IntersectionObserver advice.
- **`STYLE_PRESETS.md`:** added the missing Swiss Modern and Paper & Ink rows to the font table, corrected the JetBrains Mono source (Google Fonts), and removed a viewport `clamp()` from inside the stage.

## Philosophy

1. **You don't need to be a designer to make beautiful things.** You just need to react to what you see.
2. **Dependencies are debt.** A single HTML file will work in 10 years. A React project from 2019? Good luck.
3. **Generic is forgettable.** Every presentation should feel custom-crafted, not template-generated.
4. **Comments are kindness.** Code should explain itself to future-you (or anyone else who opens it).

## Credits

Created by [@zarazhangrui](https://github.com/zarazhangrui). The upstream project is [zarazhangrui/frontend-slides](https://github.com/zarazhangrui/frontend-slides), and the bold templates come from [zarazhangrui/beautiful-html-templates](https://github.com/zarazhangrui/beautiful-html-templates). Fork maintained by [@thefactremains](https://github.com/thefactremains).

## License

MIT. Use it, modify it, share it.

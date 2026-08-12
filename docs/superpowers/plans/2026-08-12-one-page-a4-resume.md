# One-page A4 Resume Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Produce a professional, dense, single-page A4 portrait resume PDF at Chrome 100% scale while preserving the photo, all required ByteDance evidence, and three Kaggle Silver results.

**Architecture:** Keep YAML as the content source and Liquid as the shared screen/print renderer. Add structured bullet rendering and replace the inherited print cascade with one authoritative A4 print layer; validate the generated DOM and an actual headless-Chrome PDF in an iterative loop.

**Tech Stack:** Jekyll/Liquid, YAML, SCSS, Chrome headless PDF, Poppler (`pdfinfo`, `pdftoppm`).

## Global Constraints

- A4 portrait, Chrome print scale 100%, exactly one page.
- Keep the candidate photo in print.
- Preserve all P0 ByteDance role, ownership, launch, metric, and guardrail facts.
- Preserve all specified facts and rankings for NVIDIA Nemotron, NFL Big Data Bowl 2026, and Vesuvius.
- No `transform: scale()` and no browser scale below 100%.
- Print uses one content column, horizontal section headings, white background, and compact right-aligned dates.
- Screen output must remain functional without obvious desktop or mobile regressions.

---

### Task 1: Establish reproducible build and print checks

**Files:**
- Create: `Gemfile`
- Create: `scripts/verify_resume.rb`

**Interfaces:**
- Consumes: `_data/resume.yml`, generated `_site/index.html`, and a generated PDF path.
- Produces: a nonzero exit when required content/DOM or one-page PDF conditions fail.

- [ ] **Step 1: Add a verifier that parses YAML and checks every required P0 token**

The Ruby script must use `YAML.safe_load`, assert required title/ranking/metric strings, inspect `_site/index.html` when present, and run `pdfinfo` when passed a PDF path.

- [ ] **Step 2: Run it before content/template changes**

Run: `ruby scripts/verify_resume.rb`

Expected: YAML parses; content assertions reveal any vocabulary that needs normalization rather than silently dropping facts.

- [ ] **Step 3: Add the minimal GitHub Pages-compatible Jekyll dependency declaration**

Use `github-pages` so local rendering matches the repository's deployment model.

- [ ] **Step 4: Install dependencies and build**

Run: `bundle install` and `bundle exec jekyll build`

Expected: `_site/index.html` and `_site/assets/css/main.css` are produced without YAML/Liquid/Sass errors.

- [ ] **Step 5: Capture the baseline PDF**

Run Chrome headless with `--headless --disable-gpu --no-pdf-header-footer --print-to-pdf=artifacts/resume-before.pdf file://.../_site/index.html`, then `pdfinfo artifacts/resume-before.pdf`.

Expected: baseline page count documents the current multi-page failure.

### Task 2: Restructure resume content into compact bullets

**Files:**
- Modify: `_data/resume.yml`
- Modify: `_layouts/resume.html`
- Modify: `scripts/verify_resume.rb`

**Interfaces:**
- Consumes: YAML entries with `title`, `sub`, `duration`, and optional `bullets`.
- Produces: semantic `<ul class="detail bullets"><li>…</li></ul>` markup, while retaining existing `detail`, `items`, and `summary` fallbacks.

- [ ] **Step 1: Extend verifier expectations for structured bullets and title rows**

Check generated HTML for `entry-heading`, `entry-copy`, and bullet-list classes and reject legacy “任务：/行动：/结果：” labels.

- [ ] **Step 2: Run verifier and confirm the DOM check fails**

Run: `ruby scripts/verify_resume.rb`

Expected: FAIL because the new semantic classes do not exist yet.

- [ ] **Step 3: Rewrite YAML copy at the approved information hierarchy**

Use two bullets for ByteDance, one bullet for NVIDIA, two for NFL, one for Vesuvius, a compact education entry, and compact skill lines. Preserve required metrics in `<strong>` elements and eliminate manual `<br>`-driven STAR paragraphs.

- [ ] **Step 4: Update Liquid to render compact title/date and bullet structures**

Place title/subtitle content under `.entry-copy`, date under `.entry-date`, and render `content.bullets` as list items with HTML content intentionally preserved from trusted repository YAML.

- [ ] **Step 5: Build and verify generated HTML**

Run: `bundle exec jekyll build && ruby scripts/verify_resume.rb`

Expected: PASS for YAML, required facts, links, and semantic DOM.

### Task 3: Replace the print cascade with a dedicated A4 layer

**Files:**
- Modify: `_sass/print/basic.scss`
- Modify: `_sass/print/_print.scss`
- Modify: `_config.yml`

**Interfaces:**
- Consumes: the semantic DOM from Task 2 and existing screen styles.
- Produces: one authoritative `@media print` layout with explicit `@page { size: A4 portrait; }`.

- [ ] **Step 1: Add verifier checks against forbidden compiled print patterns**

Reject `transform:scale`, a 15px print base, visible print footer rules, and gray `.detail` backgrounds in the compiled print cascade.

- [ ] **Step 2: Run build/verifier and confirm the style check fails**

Run: `bundle exec jekyll build && ruby scripts/verify_resume.rb`

Expected: FAIL on the existing 15px/global-weight print rules.

- [ ] **Step 3: Make `_print.scss` the shared reset and `basic.scss` the only media wrapper**

Remove duplicate print imports and global forced weight. Reset page/body widths, backgrounds, shadows, spacing, footer visibility, detail indentation, and word breaking.

- [ ] **Step 4: Implement the A4 single-column hierarchy**

Use compact header grid/flex with a small photo, horizontal sections, entry title/date grid, selective typography weights, tight bullet spacing, and `break-inside: avoid` on entries.

- [ ] **Step 5: Simplify print configuration defaults**

Set explicit A4-appropriate margins and remove obsolete variables/comments that encourage oversized print typography.

- [ ] **Step 6: Build and verify CSS invariants**

Run: `bundle exec jekyll build && ruby scripts/verify_resume.rb`

Expected: PASS for forbidden-pattern and DOM checks.

### Task 4: Iterate against real Chrome PDF and visual output

**Files:**
- Modify as evidence requires: `_data/resume.yml`, `_sass/print/basic.scss`, `_sass/print/_print.scss`, `_layouts/resume.html`
- Create: `artifacts/resume.pdf` (ignored from Git if appropriate)
- Create: `artifacts/resume-page.png` (ignored from Git if appropriate)

**Interfaces:**
- Consumes: built `_site/index.html`.
- Produces: a one-page PDF and full-page raster proof used for visual inspection.

- [ ] **Step 1: Generate the candidate PDF with Chrome**

Use headless Chrome with native print scale 1.0, CSS A4 page size, and no browser header/footer.

- [ ] **Step 2: Assert one page and inspect geometry**

Run: `ruby scripts/verify_resume.rb artifacts/resume.pdf` and `pdfinfo artifacts/resume.pdf`.

Expected: `Pages: 1`, A4 page size, no missing required content.

- [ ] **Step 3: Rasterize and visually inspect at high resolution**

Run: `pdftoppm -png -r 150 -singlefile artifacts/resume.pdf artifacts/resume-page` and inspect the PNG.

Check: no clipping/overflow, all projects visible, photo proportionate, ByteDance prominent, rankings/metrics scannable, no gray cards/footer, dates aligned, and no ugly technical-word breaks.

- [ ] **Step 4: Iterate in priority order until checks pass**

Reduce whitespace first, then redundant copy/DOM, then Skills/Education, and only then adjust body size within 8.5–9.5pt.

- [ ] **Step 5: Capture screen regression evidence**

Generate desktop and mobile screenshots from the built page and inspect header, sections, links, and responsive wrapping.

### Task 5: Document print workflow and complete verification

**Files:**
- Modify: `README.md`
- Modify: `.gitignore` if generated QA artifacts should remain untracked

**Interfaces:**
- Consumes: validated Chrome print settings.
- Produces: concise user instructions for recreating the accepted PDF.

- [ ] **Step 1: Update README print instructions**

Document A4 portrait, scale 100%, Chrome “Headers and footers” disabled, and explain that browser-native title/date/URL/page number cannot be disabled reliably by site CSS.

- [ ] **Step 2: Run the full verification sequence from a clean build**

Run: `bundle exec jekyll clean && bundle exec jekyll build`, YAML/DOM verifier, Chrome PDF generation, `pdfinfo`, rasterization, and final visual inspection.

Expected: build succeeds; verifier passes; PDF is one A4 page; screen screenshots show no obvious regression.

- [ ] **Step 3: Review the final diff and repository status**

Run: `git diff --check`, `git diff --stat`, and `git status --short`.

Expected: no whitespace errors and no unintended generated files staged.

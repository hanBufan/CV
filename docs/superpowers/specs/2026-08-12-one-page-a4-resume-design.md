# One-page A4 Resume Print Design

## Objective

Refactor the existing resume's print output into a single-page A4 portrait PDF at Chrome's 100% scale. The result must prioritize product manager, AI product, and payment product applications while retaining the candidate photo, ByteDance payment ownership and metrics, and all three Kaggle Silver rankings.

Print is the primary surface. The screen site remains functional and visually coherent, but it may keep its existing responsive presentation where that does not leak into print.

## Existing Rendering Path and Root Causes

The resume renders through:

`_data/resume.yml` → `_layouts/resume.html` → `assets/css/main.scss` → `_sass/resume.scss` plus `_sass/print/basic.scss` and `_sass/print/_print.scss`.

The current three-page result is caused by several compounding issues:

- `_config.yml` sets the print base size to 15px, then `basic.scss` applies medium or bold weight to nearly every text node.
- Print rules import the common print layer more than once and inherit screen layout rules with competing specificity.
- Section headings and content use the old left-rail layout, reducing line width and causing Chinese headings and English technical terms to wrap.
- Section, block, header, paragraph, detail, and list spacing stack vertically.
- `.detail` inherits a gray screen background, indentation, and aggressive Chinese `word-break: break-all` behavior.
- Titles, subtitles, and dates are separate flex rows, so dates consume extra height and can misalign.
- YAML details contain repeated “任务 / 行动 / 结果” labels and `<br>` nodes, producing long prose instead of compact resume bullets.
- The actual source selector is `.block:not(:first-child)` and is valid; an escaped-selector bug is not present.

## Chosen Approach

Keep the existing YAML, Liquid, and SCSS architecture, but create a clean, authoritative print layer and make small, targeted template/data changes. Do not create a second print-only resume template and do not use `transform: scale()` or non-100% browser scaling.

This approach keeps content and presentation maintainable, minimizes screen regressions, and gives print enough structural control to reach one page.

## Print Layout

- A4 portrait with explicit `@page` sizing and approximately 8–10mm margins, tuned against real Chrome PDF output.
- Compact two-part header: name and contact information use the main width; a small candidate photo remains visible at the right.
- One content column beneath the header.
- Horizontal section headings above their content, with a restrained divider or accent and no left title rail.
- Each entry uses a title row with CSS Grid: `minmax(0, 1fr) auto`. The title can shrink normally; the date stays right-aligned on one line.
- Subtitle/ranking metadata sits directly below the title row without a bordered date box.
- Articles avoid page breaks, but page-break rules must not create blank pages by reserving oversized blocks.
- White background, black text, no card fill, no shadow, no large padded boxes.

## Typography and Spacing

- Name: approximately 18–22pt.
- Section heading: approximately 11–13pt.
- Entry title: approximately 10–11.5pt.
- Body: approximately 8.5–9.5pt.
- Dates and metadata: approximately 8–9pt.
- Body line height: approximately 1.25–1.4.
- Use a Chinese-capable sans-serif stack without forcing a global weight. Body copy is regular; titles and metric highlights use semibold/bold selectively.
- Remove text indentation and aggressive word breaking. Prefer normal wrapping with `overflow-wrap` only where necessary.
- Reset inherited margins, padding, gaps, backgrounds, borders, and shadows inside `@media print`, then add back only deliberate spacing.

Exact values are determined by repeated Chrome rendering, not by shrinking everything uniformly.

## Content Model and Hierarchy

The YAML remains the source of truth. Details will be represented as compact resume bullets rather than explicit visual STAR labels. Liquid should render structured bullet data where useful and retain backward-compatible summary/detail handling for other entries.

Priority order is fixed:

1. Preserve all ByteDance role, scope, ownership, compliance, launch, and result requirements. Compress them into two dense bullets: ownership/execution and measurable results.
2. Preserve all required NVIDIA, NFL, and Vesuvius technologies, results, rankings, Top percentages, and Kaggle Silver labels. Each project uses one or two compact bullets.
3. Compress education to a title/date row and two short metadata lines. Courses are the first content eligible for further reduction.
4. Render Skills in two or at most three compact lines.

Metric values and the three Silver rankings receive selective strong emphasis so they can be scanned within five seconds. ByteDance remains the visually dominant entry through hierarchy and content density, not through oversized typography or background blocks.

## Header, Footer, and Chrome Behavior

- Keep the photo in both screen and print; make the print version small and rectangular/clean rather than allowing the existing large avatar dimensions.
- Hide repository-provided print and screen footers in print, including any “Made with ❤️” content and print controls.
- CSS cannot reliably disable Chrome's native date, title, URL, and page-count headers/footers. README print instructions must explicitly say: A4, portrait, scale 100%, default/no custom margins as specified by CSS, background graphics unnecessary, and “Headers and footers” off.
- The document itself must still fit the standard CSS-defined A4 content area; browser headers are not treated as resume content.

## Validation Strategy

Validation is evidence-based and iterative:

1. Parse `_data/resume.yml` with the project's available Ruby/Jekyll tooling.
2. Build the Jekyll site using the repository's supported pipeline, installing only required dependencies if missing and authorized.
3. Inspect generated HTML to confirm title/date structure, bullet rendering, contact hrefs, escaping, and absence of empty/footer elements that consume print space.
4. Serve or open the built page in headless Chrome/Chromium.
5. Generate an actual PDF using A4 portrait and 100% scale-equivalent print settings.
6. Check PDF page count programmatically and inspect a rendered full-page image visually.
7. Verify no overflow, clipping, split projects, gray blocks, footer, awkward word breaks, or isolated Skills section.
8. Compare a screen screenshot before/after or otherwise inspect desktop/mobile layouts for obvious regressions.
9. Run any existing build, lint, or test commands discovered in the repo.
10. If the PDF exceeds one page or looks cramped, iterate first on whitespace and copy density, then on Skills/Education, and only lastly make small typography adjustments.

## Scope Boundaries

- No unrelated framework migration or broad template rewrite.
- No `transform: scale()` workaround and no instruction to print below 100% scale.
- No removal of P0 ByteDance facts or required Kaggle facts.
- No attempt to spoof or programmatically control Chrome's native print-dialog header/footer preference.
- No remote push or change to the GitHub default branch unless separately requested after local implementation is complete.

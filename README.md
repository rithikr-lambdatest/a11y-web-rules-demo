# axe-core 4.12 rule violation demo

A small static site where **every page deliberately fails accessibility rules** from
`axe-core` 4.12 (`doc/rule-descriptions.md`). Point axe / a LambdaTest accessibility
scan at each page to confirm the scanner reports the expected violations.

The rules are split across pages because several are **mutually exclusive at the page
level** — e.g. you can't have both "no `lang`" and "invalid `lang`", or both "no `main`"
and "duplicate `main`", on the same page.

## Live URLs (GitHub Pages)

Hosted at **https://rithikr-lambdatest.github.io/a11y-web-rules-demo/** — point axe / a LambdaTest scan directly at these:

| Page | URL | Purpose |
|------|-----|---------|
| Index | https://rithikr-lambdatest.github.io/a11y-web-rules-demo/ | Element-level violations; page-level rules satisfied on purpose |
| All-in-one | https://rithikr-lambdatest.github.io/a11y-web-rules-demo/all-in-one.html | **One-scan URL** — 102/105 rules (uses `srcdoc` iframes) |
| aria-hidden body | https://rithikr-lambdatest.github.io/a11y-web-rules-demo/aria-hidden-body.html | `aria-hidden` on `<body>` |
| Landmarks/frames/lang | https://rithikr-lambdatest.github.io/a11y-web-rules-demo/landmarks-frames-lang.html | Duplicate landmarks, frames, invalid element `lang` |
| Invalid lang | https://rithikr-lambdatest.github.io/a11y-web-rules-demo/lang-invalid.html | Invalid `lang` value |
| Page structure | https://rithikr-lambdatest.github.io/a11y-web-rules-demo/page-structure.html | Missing `lang`/`title`/`h1`/`main`, no bypass |

For a single full-coverage scan, use **`all-in-one.html`** (requires the scanner to descend into same-origin `srcdoc` iframes; otherwise use the split pages).

## `all-in-one.html` — everything on one page (one scan)

A single self-contained page covering the same **102 of 105** rules. Element-level
violations are inline; the page-level rules that contradict each other (missing vs
invalid `lang`, 0 vs 2 `<main>`, `aria-hidden` body, etc.) are embedded as **same-origin
`srcdoc` iframes**, which axe / LambdaTest descend into. Use this when you want one URL.

> A single *document* cannot fire contradictory rules, so the iframes are how one *page*
> reaches full coverage. If a particular scanner does **not** descend into `srcdoc`
> iframes, use the split pages below instead.

## Pages & what each one violates

### `index.html` — element-level violations (the big one)
Page-level rules (lang, title, h1, main, banner, skip link) are **satisfied** here on purpose so the element-level failures are isolated.

image-alt, input-image-alt, object-alt, role-img-alt, svg-img-alt, area-alt, image-redundant-alt,
server-side-image-map, label, select-name, input-button-name, form-field-multiple-labels,
autocomplete-valid, label-title-only, avoid-inline-spacing, button-name, link-name,
link-in-text-block, summary-name, nested-interactive, label-content-name-mismatch,
aria-allowed-attr, aria-required-attr, aria-roles, aria-valid-attr, aria-valid-attr-value,
aria-required-children, aria-required-parent, aria-hidden-focus, aria-input-field-name,
aria-meter-name, aria-progressbar-name, aria-toggle-field-name, aria-tab-name, aria-tooltip-name,
aria-command-name, aria-deprecated-role, aria-prohibited-attr, aria-braille-equivalent,
aria-dialog-name, aria-treeitem-name, aria-text, aria-allowed-role, aria-roledescription,
presentation-role-conflict, focus-order-semantics, td-headers-attr, th-has-data-cells,
scope-attr-valid, empty-table-header, table-duplicate-name, table-fake-caption, td-has-header,
list, listitem, definition-list, dlitem, empty-heading, heading-order, p-as-heading, tabindex,
accesskeys, duplicate-id, duplicate-id-active, scrollable-region-focusable, hidden-content,
color-contrast, color-contrast-enhanced, blink, marquee, video-caption, no-autoplay-audio,
identical-links-same-purpose, target-size, meta-refresh, meta-refresh-no-exceptions.

### `page-structure.html` — page-level "missing" rules
No `lang`, empty `<title>`, no `<h1>`, no `<main>`, content outside landmarks, no bypass.

html-has-lang, document-title, page-has-heading-one, landmark-one-main, region, bypass,
skip-link, meta-viewport, meta-viewport-large.

### `landmarks-frames-lang.html` — duplicate landmarks, frames, invalid element lang
html-xml-lang-mismatch, landmark-no-duplicate-banner, landmark-no-duplicate-main,
landmark-no-duplicate-contentinfo, landmark-unique, landmark-banner-is-top-level,
landmark-main-is-top-level, landmark-contentinfo-is-top-level,
landmark-complementary-is-top-level, frame-title, frame-title-unique, frame-focusable-content,
frame-tested, valid-lang, duplicate-id-aria.

### `lang-invalid.html`
html-lang-valid (`<html lang="xyzzy">`).

### `aria-hidden-body.html`
aria-hidden-body (isolated because it hides the whole page from AT).

## ⚠️ Many of these rules are DISABLED by default in axe-core
The core scan (`wcag2a, wcag2aa, wcag21a, wcag21aa, best-practice`) will **not** report:

- **WCAG 2.2:** `target-size`
- **WCAG AAA:** `color-contrast-enhanced`, `identical-links-same-purpose`, `meta-refresh-no-exceptions`
- **Experimental:** `p-as-heading`, `table-fake-caption`, `td-has-header`, `focus-order-semantics`, `hidden-content`, `label-content-name-mismatch`, `css-orientation-lock`
- **Deprecated:** `aria-roledescription`, `audio-caption`, `duplicate-id`, `duplicate-id-active`, `landmark-complementary-is-top-level`

To catch them, enable the relevant tags/rules in your run options, e.g.:
```js
axe.run(document, {
  runOnly: { type: 'tag', values: ['wcag2a','wcag2aa','wcag21a','wcag21aa','wcag22aa','wcag2aaa','best-practice','experimental','deprecated'] }
});
```
For a **LambdaTest** scan, enable `wcagVersion: wcag22aa`, `bestPractice: true`, and `needsReview: true` to surface the largest set.

## How to scan

Serve the folder (some rules need a real origin, not `file://`):
```bash
cd a11y-rules-demo
python3 -m http.server 8080
# then open http://localhost:8080/index.html
```
Then scan with any of:
- **axe browser extension** (Chrome/Firefox) — enable experimental rules in its settings for full coverage.
- **@axe-core/cli**: `npx @axe-core/cli http://localhost:8080/index.html`
- **LambdaTest accessibility scan** against the served/hosted URL.

## Rules intentionally not included (hard to trigger statically)
- `aria-conditional-attr` — needs a role whose allowed attrs depend on another attr's state.
- `css-orientation-lock` — needs a CSS transform that locks orientation across a media query.
- `audio-caption` (deprecated) — add an `<audio>` with no `<track kind="captions">` if you need it.

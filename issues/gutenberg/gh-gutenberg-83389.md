# #83389: Tab Panel: Add link, heading, and button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`f8eb176`](https://github.com/WordPress/gutenberg/commit/f8eb176f4e2dc61455d343d6236fbc77e1499f04)
- **Discussion:** [#83389](https://github.com/WordPress/gutenberg/pull/83389) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tab Panel block (`core/tab-panel`) now declares `color.link`, `color.heading`, and `color.button` in its `block.json` supports, exposing link, heading, and button colour controls in the Global Styles editor and the block inspector. The underlying CSS via `style.elements` already rendered on the front end; this change adds the missing UI controls, bringing the block in line with other core blocks as part of the design-tools consistency effort (related to #43241). No PHP or render-callback changes were required.

## Impact

- **Theme & block developers:** The Tab Panel block now accepts element-level colour overrides for `link`, `heading`, and `button` through Global Styles and the block inspector. If you previously applied these colours via custom CSS targeting the panel's wrapper, the new controls provide a first-class alternative. No code changes are required to benefit.
- **Site owners / editors:** New colour pickers appear under Global Styles → Blocks → Tab Panel → Elements and in the block inspector's Elements section. Mobile-responsive values are supported.
- **No breaking changes.** The block's existing `color.background` and `color.text` supports are unchanged. No PHP, REST, or database changes.

## Technical details

The diff modifies three files:

1. **`packages/block-library/src/tab-panel/block.json`** — three keys added to the existing `color` supports object:

```json
"color": {
  "background": true,
  "button": true,
  "heading": true,
  "link": true,
  "text": true,
  "__experimentalDefaultControls": {
    "background": true,
    ...
  }
}
```

2. **`packages/block-library/src/tab-panel/README.md`** — regenerated to list the three new sub-keys under the `color` support entry.

3. **`docs/reference-guides/core-blocks/README.md`** — the Tab Panel row's supports string updated from `color (background, text)` to `color (background, button, heading, link, text)`.

No changes to `render.php`, `index.js`, or any Interactivity API module. The PR description notes that the render callback's Tag Processor only sets `id`, `aria-labelledby`, and Interactivity API attributes, so the wrapper element's class and inline styles (which carry the `style.elements` CSS) pass through untouched. The front-end rendering of element colours was already functional; only the editor-side controls were missing.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no recorded design debate or rejected alternatives. Merged as `f8eb176`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

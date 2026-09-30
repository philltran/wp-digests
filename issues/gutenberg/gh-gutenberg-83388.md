# #83388: Tab Panels: Add button colour support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`eebc201`](https://github.com/WordPress/gutenberg/commit/eebc201131311329e5a5e7bb0164792e6fef946f)
- **Discussion:** [#83388](https://github.com/WordPress/gutenberg/pull/83388) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tab Panels block (`core/tab-panels`) now declares `color.button` in its `block.json` supports, enabling button colour controls in Global Styles and the block inspector. The block already supported background, text, heading, and link colours; button was the missing element. Because Tab Panels is a static block whose wrapper is produced by `useBlockProps.save()`, no PHP render change was required — element colours under `style.elements` already render on the front end once the support is declared.

## Impact

- **Site owners / editors:** A new "Button" colour control appears under Global Styles → Blocks → Tab Panels → Elements, and in the block inspector's Styles → Elements panel. No migration or configuration change needed.
- **Plugin & theme developers:** No code changes required. If you build custom styles targeting `.wp-block-tab-panels button`, the new support simply makes those styles settable through the UI. No new hooks, filters, or REST schema fields are introduced.
- **No action required** for existing sites; the change is purely additive to the block's declared supports.

## Technical details

The diff is confined to three files:

1. **`packages/block-library/src/tab-panels/block.json`** — adds `"button": true` to the existing `color` supports object, alongside the already-present `background`, `text`, `heading`, and `link` keys.

2. **`packages/block-library/src/tab-panels/README.md`** — regenerated to list `button: true` under the colour supports section.

3. **`docs/reference-guides/core-blocks/README.md`** — regenerated; the Tab Panels entry's supports line changes from `color (background, heading, link, text)` to `color (background, button, heading, link, text)`.

No changes to `render.php`, `index.js`, or any PHP file. The PR description notes that element colours on a block's `style.elements` already render on the front end without the support declaration; the support flag is what surfaces the control in the editor UI (Global Styles and block inspector). The `__experimentalDefaultControls` object in `block.json` is untouched.

## Contribution

Opened by @aaronrobertshaw as part of the design-tools consistency work tracked in #43241. The PR body notes it was "implemented, built and screenshotted by a Claude Code agent from a predefined task." Co-authored with @shail-mehta. The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

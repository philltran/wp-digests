# #83595: Tabs: Add minimum width support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Tabs`
- **Merged:** [`b2a3f12`](https://github.com/WordPress/gutenberg/commit/b2a3f12a35a4f2f4a80031a475c4477d05a470ac)
- **Discussion:** [#83595](https://github.com/WordPress/gutenberg/pull/83595) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block (`core/tabs`) gains `dimensions.minWidth` support, letting developers set a minimum width on the entire tabbed interface through Global Styles or the block inspector. This closes a gap in the block's design controls (it previously supported align, background, colour, spacing, shadow, and typography but no dimensions) as part of the broader design-tools consistency effort tracked in #43241. No PHP changes were required because the existing render callback already passes `class` and `style` through untouched.

## Impact

- **Site owners / editors:** A new "Minimum width" field appears under Dimensions in the Tabs block inspector and in Global Styles → Blocks → Tabs. No migration or configuration change is needed.
- **Plugin & theme developers:** No code changes required. The support is declarative in `block.json`; any theme or plugin that already handles `dimensions.minWidth` on other blocks will pick this up automatically. No new hooks, filters, or REST schema changes.
- **Headless / REST consumers:** The serialized block markup gains a `min-width` inline style on the wrapper element when the value is set. No new REST fields.
- **No action required** for existing sites; the feature is opt-in.

## Technical details

The diff adds a single key to the `supports` object in `packages/block-library/src/tabs/block.json`:

```json
"dimensions": {
    "minWidth": true
}
```

The wrapper element is produced in `save.js` via `useBlockProps.save()`, which serializes the `min-width` CSS custom property into the saved markup. The PHP render callback post-processes that markup with `WP_HTML_Tag_Processor` solely to set `data-wp-*` attributes and leaves `class` and `style` untouched, so no PHP file was modified.

Minimum height is explicitly excluded. The PR description explains that on Tabs it would have to cover the tab row, the gap, and the panel area, and the tab row's height varies with font size, button padding, and wrapping. The author states it belongs on Tab Panels, where it applies to the panel area directly.

Two documentation files were regenerated to list the new support:
- `docs/reference-guides/core-blocks/README.md` — the Tabs entry now reads `dimensions (minWidth)` in its supports list.
- `packages/block-library/src/tabs/README.md` — a new `dimensions` → `minWidth: true` entry was added under the supports section.

## Contribution

Opened by @aaronrobertshaw with @ramonjd as co-author. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The record contains no human review discussion — the only two comments are automated bot posts (performance metrics and a flaky-test report). No design debate or rejected alternatives are visible in the thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

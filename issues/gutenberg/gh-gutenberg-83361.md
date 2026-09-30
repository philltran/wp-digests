# #83361: Tabs: Add border support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Tabs`
- **Merged:** [`ee2492f`](https://github.com/WordPress/gutenberg/commit/ee2492f1882d3825759dbaf77a3dc0fca1cff1ea)
- **Discussion:** [#83361](https://github.com/WordPress/gutenberg/pull/83361) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Tabs block gains border support (color, radius, style, and width) through the `__experimentalBorder` key in its `block.json`. Previously the block exposed background, colour, shadow, spacing, and typography controls, but had no way to draw a frame around the entire tabbed interface — the Tab List's own border only styles individual tab buttons. No PHP changes were required because the block's render callback already preserves `class` and `style` attributes on the wrapper element via `WP_HTML_Tag_Processor`.

## Impact

- **Site owners / theme developers:** Border controls for the Tabs block are now available in Global Styles (Blocks → Tabs → Border) and in the block inspector, with full responsive (mobile/desktop) support. No code changes needed.
- **Plugin & theme developers:** No new public API, hook, or REST schema change. The addition is a standard block-supports key; any code that reads `block.json` supports will see the new `__experimentalBorder` entry.
- **No action required** for existing sites or plugins. The change is purely additive and backward-compatible.

## Technical details

The entire diff is a single addition to `packages/block-library/src/tabs/block.json`, inserting the following into the `supports` object:

```json
"__experimentalBorder": {
  "color": true,
  "radius": true,
  "style": true,
  "width": true
}
```

No PHP file was modified. The Tabs block's render callback uses `WP_HTML_Tag_Processor` to build the wrapper element and already retains its `class` and `style` attributes, so the border CSS emitted by the block-supports pipeline flows through without further work.

Two behavioral notes from the PR description:

- The docs generators skip `__experimental`-prefixed keys, so no generated documentation changes accompany this PR.
- The Tabs block has no default padding, so the tab list and panel content sit flush against the new border. Padding is already available as a separate block control if spacing is needed.

## Contribution

Opened by @aaronrobertshaw with co-authorship from @ramonjd. The PR carries only 2 comments and 0 reactions, indicating a straightforward, uncontested merge. The author notes the implementation was produced by a Claude Code agent from a predefined task as part of the broader design-tools consistency effort tracked in #43241.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

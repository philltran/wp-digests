# #83055: Playlist: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `Global Styles`, `[Feature] Design Tools`, `[Block] Playlist`
- **Merged:** [`3d20e65`](https://github.com/WordPress/gutenberg/commit/3d20e65baec96ef20d32d26508917d50475fa67e)
- **Discussion:** [#83055](https://github.com/WordPress/gutenberg/pull/83055) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Playlist block (`core/playlist`) now declares the `shadow` block support in its `block.json`, enabling box-shadow styling via Global Styles and the block inspector. This closes a gap where the block already supported color, border, spacing, and typography but lacked shadow control, as part of the broader design-tools consistency effort tracked in #43241. No PHP or JavaScript changes were required because the existing `useBlockProps.save()` serialization and the server-render `WP_HTML_Tag_Processor` pass already handle the `figure` wrapper's `class` and `style` attributes.

## Impact

- **Site owners / theme developers:** The Playlist block now appears under Blocks → Playlist → Shadow in the Global Styles editor. A shadow preset (e.g. Natural, Deep) can be applied globally or overridden per block instance in the inspector. No code changes needed.
- **Plugin & theme developers:** If you build custom Playlist block variants or extend the block's `block.json`, be aware that `shadow` is now part of its declared supports. No migration or configuration change is required for existing themes.
- **Headless / REST consumers:** No change to the REST schema or serialized block markup structure; the shadow is emitted as a standard `style` attribute on the `figure` element, same as other supported blocks.
- **No action required** for any existing implementation.

## Technical details

The diff is confined to three files:

1. **`packages/block-library/src/playlist/block.json`** — adds `"shadow": true` to the `supports` object, alongside the existing `color`, `interactivity`, `spacing`, and `typography` entries.
2. **`packages/block-library/src/playlist/README.md`** — regenerated to list the new `shadow` support with a link to the block-supports reference.
3. **`docs/reference-guides/core-blocks/README.md`** — regenerated; the Playlist row's supports list now reads `align, anchor, color (background, gradients, link, text), interactivity, shadow, spacing (margin, padding), typography (fontSize)`.

No changes to `render.php`, `index.js`, or any PHP file. The shadow is serialized onto the `figure` wrapper by `useBlockProps.save()` (the standard block-supports pipeline). The server-side render in `render.php` uses `WP_HTML_Tag_Processor` to inject only `data-wp-interactive` and `data-wp-context` attributes on that `figure`, leaving its `class` and `style` untouched, so the client-serialized shadow styles pass through to the front end without modification.

## Contribution

Opened and merged by @aaronrobertshaw with co-authorship from @jorgefilipecosta. The PR body notes the implementation, build, and screenshots were produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no design debate or rejected alternatives recorded.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

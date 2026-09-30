# #83045: Heading: Add shadow support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aaronrobertshaw
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Heading`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`3e78a97`](https://github.com/WordPress/gutenberg/commit/3e78a9750aae3fb0ce22a54f87361b5b4d3da0e2)
- **Discussion:** [#83045](https://github.com/WordPress/gutenberg/pull/83045) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Heading block now supports the `shadow` block support, allowing box-shadow to be applied via Global Styles or the block inspector. This closes a gap where Heading already supported color, border, spacing, and typography but lacked shadow control, as part of the broader design-tools consistency effort tracked in #43241. The change is a single-line addition to the block's `block.json` supports; serialization and rendering are handled entirely by the existing block-supports infrastructure.

## Impact

- **Site owners / editors:** Heading blocks can now receive a `box-shadow` through Global Styles (Blocks → Heading → Shadow) or per-instance in the block inspector. No code or configuration changes required.
- **Plugin & theme developers:** No action required. The `shadow` support is a standard block support; any theme or plugin that already handles `wp-block-heading` styling will pick up the shadow automatically via the serialized `style` attribute. No new hooks, filters, or REST schema fields are introduced.
- **Hosting & platform:** No migration, no DB change, no PHP change. The server-side render path (`WP_HTML_Tag_Processor`) already preserves the `style` attribute on the heading element.
- **Headless & REST consumers:** The serialized block markup will include an inline `style` attribute with `box-shadow` when a shadow is set. No new REST fields or attributes are exposed.

## Technical details

The functional change is a single addition to `packages/block-library/src/heading/block.json`:

```json
"supports": {
  "color": { /* …existing… */ },
  "shadow": true,
  "spacing": { /* …existing… */ }
}
```

With `shadow` declared in `supports`, the block-supports system handles the rest:

- **Editor serialization:** `useBlockProps.save()` emits the `box-shadow` value into the element's `style` attribute when a shadow is selected in the inspector or via Global Styles.
- **Server render:** The Heading block's `render.php` uses `WP_HTML_Tag_Processor` to add the `wp-block-heading` class. It does not strip or rewrite the `style` attribute, so the serialized shadow passes through to the front end without any PHP modification.
- **Documentation:** `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/heading/README.md` are regenerated to list `shadow` among the block's supports.

No new attributes, hooks, filters, or REST schema entries are introduced. The change is purely declarative in `block.json`.

## Contribution

Opened by @aaronrobertshaw and co-authored with @talldan. The PR body notes the implementation was produced by a Claude Code agent from a predefined task. The discussion is minimal (2 comments, 0 reactions) with no visible design debate or rejected alternatives; the approach of simply declaring the existing `shadow` support in `block.json` was the straightforward path given the infrastructure already in place.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

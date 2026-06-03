# #78064: Image block: Add "Mark as decorative" toggle for accessibility

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @SteveJonesDev
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `Needs Design Feedback`, `[Package] Block library`, `[Block] Image`, `First-time Contributor`
- **Merged:** [`99bc2d5`](https://github.com/WordPress/gutenberg/commit/99bc2d5f5c82ef8e8cc8bb713e828cb8b5314c1b)
- **Discussion:** [#78064](https://github.com/WordPress/gutenberg/pull/78064) · 16 comments · 0 reactions

## Summary

Adds an explicit "Mark as decorative" toggle to the Image block inspector, providing a machine-readable signal for purely decorative visuals. When enabled, the block clears alt text, captions, and links, and appends `role="none"` to the `<img>` element to ensure assistive technologies skip the media.

## Impact

- **Block & Theme Developers**: The `core/image` block now includes an `isDecorative` boolean attribute in its JSON metadata. Existing themes or plugins parsing Image block data should accommodate this new property, though it defaults to `false` and requires no migration.
- **Editors & Site Owners**: A new checkbox appears in the Image block's Content inspector under Media. Enabling it hides alt text, caption, and link controls to prevent conflicting markup. Lightbox and decorative status are mutually exclusive; enabling lightbox automatically resets the toggle.

## Technical details

- Registers `isDecorative` (type: `boolean`, default: `false`) in `packages/block-library/src/image/block.json`.
- In `image.js`, the toggle is rendered via `<CheckboxControl>` inside a `<ToolsPanelItem>`. The `updateIsDecorative()` setter clears `alt`, `caption`, `href`, and link attributes. Caption rendering and link toolbars are gated by `{ !isDecorative && ... }` conditionals.
- The block editor preview replaces the empty alt with a descriptive fallback string (e.g., `"This image has been marked as decorative; its file name is..."`) so screen readers in the editor retain context.
- `save.js` conditionally applies `role={ isDecorative ? 'none' : undefined }` to the `<img>` tag during frontend save and server-side rendering.
- Test fixtures across `packages/block-library/src/*/fixtures/blocks/` are updated to include `"isDecorative": false`.

## Contribution

Authored by @SteveJonesDev (first-time contributor) with co-authors @t-hamano, @jameskoster, @joedolson, and @joen. The design discussion debated auto-detecting decorative status from empty alt text but was rejected to prevent accidental silencing of meaningful images during edits. Review feedback led to mutual exclusivity with lightbox/captions/links, a destructive-action confirmation dialog, and tightened UI copy for brevity. Merged following final review cycles.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

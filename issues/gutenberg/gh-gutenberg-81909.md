# #81909: Try adding a Grid block variation to Gallery

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Gallery`
- **Merged:** [`a08fc3e`](https://github.com/WordPress/gutenberg/commit/a08fc3ea30b4792743e2451758397b188745fdb0)
- **Discussion:** [#81909](https://github.com/WordPress/gutenberg/pull/81909) · 3 comments · 3 reactions
- **Usefulness:** 4/5

## Summary

The Gallery block gains a Grid layout alongside its existing custom Flex layout, selectable through the block's variation switcher in the inspector. In Grid mode the gallery's children are sized with the standard child-sizing controls, and Flex-only controls (Columns, Crop images to fit) are hidden. Existing galleries, and any with missing or malformed `layout` data, continue to render as Flex. The PR closes #42240 and is expected to fix #55231.

## Impact

- **Site owners / editors:** Galleries can be switched between Flex (default) and Grid, and child images can be resized in Grid mode. Existing galleries keep their current appearance.
- **Plugin & theme developers:**
  - `core/gallery` `supports.layout` changes: `allowEditing` goes from `false` to `true`, and `allowSizingOnChildren: true` is added. `allowOrientation`, `allowJustification`, `allowVerticalAlignment` and `allowWrap` are set to `false`. Code that reads these supports or filters `block_type_metadata` for gallery may see different values.
  - Themes and CSS targeting `.wp-block-gallery` in the editor should account for `is-layout-flex` versus `is-layout-grid`. Grid galleries do not get `columns-*` or `is-cropped` classes.
  - Dynamic gallery output differs by layout type: only Flex galleries get the `--wp--style--unstable-gallery-gap` per-instance class and style.
- **Headless / REST consumers:** Grid galleries store `layout.type = 'grid'` (with `columnCount` / `minimumColumnWidth`) in the block attributes. Consumers that assume flex may need to handle it.
- No migration is required for existing content.

## Technical details

**block.json** (`packages/block-library/src/gallery/block.json`): `supports.layout` now has `allowEditing: true`, `allowSizingOnChildren: true`, and `allowOrientation`, `allowJustification`, `allowVerticalAlignment`, `allowWrap` all `false`. The default stays `{ "type": "flex" }`, and `allowSwitching` / `allowInheriting` remain `false`. The README and core-blocks reference docs are updated to match.

**edit.jsx**:
- Imports the new helpers `isGalleryFlexLayout` and `isObject` from `./shared` and derives `isFlexLayout` from the `layout` attribute.
- A `useEffect` tracks the previous layout in a `useRef`. When `layout.type` changes and the new layout lacks `columnCount` or `minimumColumnWidth`, it copies them from the previous layout. It calls `__unstableMarkNextChangeAsNotPersistent()` followed by `setAttributes( { layout } )`, so switching back restores Grid settings.
- The `columns-*`, `columns-default` and `is-cropped` classes, the Columns and Crop images to fit `ToolsPanelItem`s, the `resetAll` writes to `columns` / `imageCrop`, and `GalleryGapCustomProperties` are all gated on `isFlexLayout`.

**gallery.jsx**: the same class gating is applied to the wrapper `<figure>`.

**editor.scss**: the Flex-specific rules are scoped to `figure.wp-block-gallery.is-layout-flex`. A new `.wp-block-gallery.is-layout-grid` rule makes `.blocks-gallery-caption`, `.components-placeholder` and `.block-editor-media-placeholder` span `grid-column: 1 / -1`.

**index.php** (`block_core_gallery_render`): computes `$is_flex_layout`, which is true when `$attributes['layout']` isn't an array, has no string `type`, has an empty `type`, or has `type === 'flex'`. For dynamic galleries the `columns-*` / `is-cropped` wrapper classes are only added for Flex. The `wp_unique_id( 'wp-block-gallery-' )` class, the `--wp--style--unstable-gallery-gap` computation and the viewport media-query gap styles are moved inside an `if ( $is_flex_layout )` block. The diff is truncated after that point, so the remaining PHP and any variation registration files are not visible here.

The PR description also refers to a variation UI for switching layouts, but the variation registration code falls in the truncated portion of the diff.

## Contribution

Authored by @tellthemachines, who disclosed using Codex/GPT for the implementation. The PR was framed as a "try" and closes a long-standing request (#42240). The record shows no notable design debate; the visible discussion is bot output (bundle size, a flaky e2e test, props).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

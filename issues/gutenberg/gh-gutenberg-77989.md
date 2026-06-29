# #77989: Add layout and block spacing support to Latest Posts block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Latest Posts`, `[Feature] Layout`
- **Merged:** [`98f2b7f`](https://github.com/WordPress/gutenberg/commit/98f2b7ff6c750fd072a5d0caf9198de8147d66ea)
- **Discussion:** [#77989](https://github.com/WordPress/gutenberg/pull/77989) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Latest Posts block (core/latest-posts) now adopts the standard layout and spacing.blockGap block supports, replacing its legacy postLayout and columns attributes. Existing instances are automatically migrated to the new schema via a deprecation layer, preserving grid layouts and column counts while unifying the block’s editing controls and rendering classes with core’s native layout system.

## Impact

- **Theme/Block Developers:** The `postLayout` and `columns` attributes are deprecated in favor of `layout.type` and `layout.columnCount`. If you query or programmatically modify Latest Posts blocks, update your code to read/write the `layout` structure instead.
- **Platform/Hosting Teams:** No action required. Existing content is backward-compatible via automatic migration on edit.
- **Editors:** No action required. Grid/List toggle and column controls are now managed through the Layout panel with native responsive behavior enabled by default.

## Technical details

- **block.json**: Removed `postLayout` (string) and `columns` (number) from the current attributes schema. Added `"layout": true` to supports, and configured `spacing.blockGap` with a default of `1.25em` alongside experimental controls.
- **deprecated.js**: Introduced migration v1 that converts legacy `{ postLayout: 'grid', columns: 4 }` to `{ layout: { type: 'grid', columnCount: 4 } }`. Preserves other legacy attributes during the migration pass.
- **edit.js**: Destructures `__unstableLayoutClassNames` and passes it to `useBlockProps`. Reads `layout` from block context, falling back to legacy `postLayout`/`columns` for initial state. Replaced custom ToolsPanel UI with standard layout inspector controls.
- **index.php**: The `render_block_core_latest_posts()` function now prioritizes `$attributes['layout']['type']`, falling back to `postLayout`. Emits `is-grid`, `columns-{N}`, and `has-native-responsive-grid` CSS classes based on the resolved layout config.
- **style.scss**: Scoped legacy flex-based grid rules behind `&:not(.is-layout-grid)`. Added a temporary media query fallback for responsive columns when native layout grid is active but not yet applied.

**Before (legacy attribute pattern):**
```json
{
  "postLayout": "grid",
  "columns": 4
}
```

**After (standard layout support):**
```json
{
  "layout": {
    "type": "grid",
    "columnCount": 4,
    "minimumColumnWidth": null
  }
}
```

## Contribution

Opened and merged by @tellthemachines with review guidance from @ramonjd. The PR standardizes the block’s layout behavior by aligning it with core’s layout support system. During review, authors confirmed that legacy `postLayout`/`columns` attributes should be moved to a deprecation layer for automatic migration rather than maintained in parallel, following recent patterns in Query-related pull requests. Merged at commit 98f2b7f.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

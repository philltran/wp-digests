# #80505: State control - avoid tertiary variant on toggle to match style of other dropdown toggles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`15fda24`](https://github.com/WordPress/gutenberg/commit/15fda24445632b5d3db78e73c4f03d530aa9e1fb)
- **Discussion:** [#80505](https://github.com/WordPress/gutenberg/pull/80505) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor's state control dropdown toggle now matches the visual style of other dropdown toggles by removing the `tertiary` variant and adjusting the button size. Previously, the toggle displayed a pale blue background on hover and used a `compact` size, which conflicted with the tools panel and other dropdown controls. This change aligns the hover, focus, and sizing behavior across the block editor's UI components.

## Impact

- **Block editor users & theme/plugin developers:** No code changes required. The change is purely visual within the block editor interface.
- **Developers extending block editor UI:** If you are directly styling or overriding the `StateControl` component, note that the internal button now defaults to no variant and `size: 'small'` (or `compact` when `showText` is true). CSS targeting the tertiary hover background will no longer apply.
- **No action required** for standard block development, theme styling, or plugin integration.

## Technical details

The change modifies `packages/block-editor/src/components/global-styles/state-control.js`. The `toggleProps` object previously passed `variant: 'tertiary'` and `size: 'compact'` to the underlying button component. The diff removes the `variant` prop entirely and updates the fallback size to `'small'`:

```diff
-        ? { size: 'compact', variant: 'tertiary', iconPosition: 'right' }
-        : { size: 'compact', variant: 'tertiary' };
+        ? { size: 'compact', iconPosition: 'right' }
+        : { size: 'small' };
```

This eliminates the tertiary button's hover background color and aligns the focus ring and dimensions with other dropdown toggles in the block editor. No new hooks, filters, REST schema fields, or block.json properties are introduced.

## Contribution

Opened to resolve a minor UI inconsistency spotted during development, the fix required only removing the `tertiary` variant and adjusting the size prop to match existing dropdown patterns. The change was co-authored with @ramonjd, quickly reviewed, and merged. It was subsequently cherry-picked to both the `wp/7.1` and `release/23.6` branches to ensure inclusion in upcoming releases.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

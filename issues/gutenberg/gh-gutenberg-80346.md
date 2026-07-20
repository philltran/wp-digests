# #80346: Device preview dropdown: use active color for device icon when responsive styles are active

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Style States`
- **Merged:** [`105e544`](https://github.com/WordPress/gutenberg/commit/105e5440060692f01238acf5060d6c01d048c7c8)
- **Discussion:** [#80346](https://github.com/WordPress/gutenberg/pull/80346) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor now applies a purple tint to the device preview dropdown toggle when responsive styles editing is active. This change adds an `is-responsive-editing` modifier class to the dropdown component and styles the toggle using the existing `--wp-block-synced-color` CSS variable. It provides a persistent visual cue in the editor header so users know they are in responsive editing mode.

## Impact

- **Theme & plugin developers**: No action required. This is an internal editor UI enhancement with no public API changes, hooks, or filters affected.
- **Site owners & editors**: Improved visual feedback in the block editor when toggling responsive styles.
- **Hosting & platform teams**: No configuration or migration needed.

## Technical details

The diff modifies `packages/editor/src/components/preview-dropdown/index.js` to conditionally apply the `is-responsive-editing` class to the `DropdownMenu` wrapper via `clsx`, driven by the `isResponsiveEditing` state. In `packages/editor/src/components/preview-dropdown/style.scss`, a new rule targets `.editor-preview-dropdown.is-responsive-editing .editor-preview-dropdown__toggle` and sets `color: var(--wp-block-synced-color);`. The device icon inherits this color via `currentColor`, and the "View" text label (when the "Show button text labels" preference is enabled) is tinted identically. No JavaScript APIs, REST routes, or block.json fields are introduced or modified.

## Contribution

Opened by @ramonjd with co-authors @tellthemachines and @noruzzamans. During review, @tellthemachines suggested applying the color to the button wrapper to ensure the "View" text label also tints when text labels are enabled; the author implemented this in the same PR. The change was backported to the `wp/7.1` branch for inclusion in the upcoming release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #79966: Media editor: address accessibility review feedback

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `[Feature] Media`
- **Merged:** [`a3c26b3`](https://github.com/WordPress/gutenberg/commit/a3c26b3d7c059f9e9be75b2bdd2fcb5c75b64db6)
- **Discussion:** [#79966](https://github.com/WordPress/gutenberg/pull/79966) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request refines screen reader announcements in the Gutenberg media editor to improve assistive technology comprehensibility. Crop adjustments now report pixel dimensions instead of percentages, rotation announcements include directional context (clockwise/counterclockwise), and the `aria-live` region suppresses unchanged values like zoom when only cropping or rotating. Resize handle labels were also updated to clarify interaction intent.

## Impact

- **Site editors**: Improved screen reader feedback when cropping, rotating, or resizing images in the native media editor.
- **Plugin & theme developers**: No code changes required. This is a client-side editor enhancement that does not expose new hooks, filters, or REST endpoints.
- **Accessibility testers**: Verify that crop resize handle `aria-label` strings now include "from" (e.g., `Resize from top-left corner`) and that live region announcements match the new pixel/direction format.
- No action required for existing codebases or configurations.

## Technical details

The diff modifies `packages/media-editor/src/image-editor/react/components/stencils/rectangle-stencil.tsx` to update the `getHandleLabel()` function, changing all eight resize handle `aria-label` strings from `"Resize [position]"` to `"Resize from [position]"`. The core behavior change is validated in `packages/media-editor/src/image-editor/react/hooks/test/use-aria-announcer.ts`, which confirms that crop announcements now calculate pixel dimensions using `naturalWidth` and `naturalHeight` rather than percentage-based thresholds. Rotation announcements were adjusted to report the visual direction (e.g., `"Rotated 15 degrees clockwise"`) and prioritize rotation over zoom in combined announcements. The hook also suppresses unchanged state values when only crop or rotation changes occur.

Before/after for resize handle labels:
```diff
- return __( 'Resize top-left corner' );
+ return __( 'Resize from top-left corner' );
```

## Contribution

Opened by @ramonjd and merged following review cycles with @andrewserong and @talldanwp. Early testing revealed that rotation announcements were incorrectly normalizing values, causing unexpected directional reports; the author adjusted the logic to report the visual rotation value directly. @andrewserong later identified a threshold mismatch in the crop announcement guard that missed 1px adjustments, which was resolved by aligning the check and announcement regions. The PR explicitly excludes modal `inert` handling to keep scope focused, deferring it to #79969.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

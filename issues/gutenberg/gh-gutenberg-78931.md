# #78931: Media Editor: Fix sidebar overflowing the modal between the small and medium breakpoints

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Media`
- **Merged:** [`9d43692`](https://github.com/WordPress/gutenberg/commit/9d43692e67a4fb7f62ac526dbd2b565c9d88d9ab)
- **Discussion:** [#78931](https://github.com/WordPress/gutenberg/pull/78931) · 3 comments · 1 reactions

## Summary

Fixes a layout regression where the Media Editor's details/crop sidebar overflows its modal container at viewport widths between ~600px and 782px. The overflow occurs because `@wordpress/interface`’s `ComplementaryArea` applies `width: 100vw` in mobile mode, exceeding the inset bounds of a `size="fill"` modal. This PR introduces a scoped CSS override to constrain the sidebar to the modal's width while preserving the existing overlay behavior when the panel is toggled open.

## Impact

- **Media Editor users & frontend developers:** No code changes or configuration updates required. The fix applies automatically upon updating WordPress.
- **Plugin/theme developers:** If you host the Media Editor inside a custom modal using `size="fill"`, the sidebar will now correctly respect your modal's width breakpoint. No migration is needed for existing implementations.
- **Headless & REST consumers:** No impact; this is a purely client-side block-editor UI adjustment with no API, schema, or backend changes.

## Technical details

- The fix adds a CSS rule scoped to `.media-editor` that activates below the `medium` breakpoint (`<782px`).
- It overrides two inline `width: 100vw` declarations applied by `@wordpress/interface`'s panel wrapper using `!important`, changing it to `width: 100%` to match the modal's inset container.
- A `:has(.interface-complementary-area__fill)` pseudo-class constraint ensures the width is only applied when the skeleton sidebar wrapper is open, preventing an empty white backdrop from covering the canvas when closed.
- Files modified: `build/styles/editor/style.css` and `style-rtl.css` (+150 B total). No changes were made to `@wordpress/interface` or other editor shells (edit-post, edit-site), keeping the fix isolated to the media modal consumer.

## Contribution

PR #78931 was authored by @ramonjd and merged as commit `9d43692`, with co-authorship noted for @andrewserong. The work addresses issue #78896, which originally reported the viewport-specific sidebar overflow. Review feedback confirmed that a scoped `.media-editor` CSS override was necessary to avoid modifying shared `@wordpress/interface` internals or inadvertently affecting other editor contexts. No alternative layout approaches were merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

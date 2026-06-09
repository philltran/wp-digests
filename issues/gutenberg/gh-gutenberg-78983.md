# #78983: Tooltip: Use md border radius for portaled popups.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `Design System`, `[Package] UI`
- **Merged:** [`cca80fe`](https://github.com/WordPress/gutenberg/commit/cca80fecc4261fba24f6d1ae40bce7b463027ad8)
- **Discussion:** [#78983](https://github.com/WordPress/gutenberg/pull/78983) · 4 comments · 0 reactions

## Summary

Updates the border radius of portaled tooltip surfaces from `sm` (2px) to `md` (4px) across both `@wordpress/components` and `@wordpress/ui`. This aligns tooltips with other overlay surfaces like menus and popovers, which now use the `md` token to establish a clear visual hierarchy between triggers and floating UI. Native tooltip styling already matched this value, so the change standardizes custom portaled overlays.

## Impact

- **Plugin & theme developers**: No code changes required. The rendered border radius of portaled tooltips shifts from 2px to 4px. Trigger elements (buttons, icon buttons) retain the `sm` radius unchanged.
- **Block editor & admin interface consumers**: Minor visual update; floating UI surfaces now visually distinguish themselves from compact controls per the Design System hierarchy.
- **Hosting, platform, & REST/headless consumers**: Not affected. This is a frontend CSS/SCSS token change scoped to React UI components.

## Technical details

The diff swaps small border radius tokens for medium ones in two design system style files:

- `packages/components/src/tooltip/style.scss`: Replaces `$radius-small` with `var(--wpds-border-radius-md)` on the `.components-tooltip` class.
- `packages/ui/src/tooltip/style.module.css`: Replaces `var(--wpds-border-radius-sm)` with `var(--wpds-border-radius-md)`.

Before/after pattern:
```scss
/* Before */
border-radius: $radius-small;

/* After */
border-radius: var(--wpds-border-radius-md);
```

The change is purely a design system token substitution. No JavaScript logic, React props, ARIA attributes, or component APIs are modified. Changelogs for both `@wordpress/components` and `@wordpress/ui` are updated to reflect the enhancement.

## Contribution

Opened by @jameskoster and co-authored by @ciampo and @juanfra. The change addresses a design hierarchy gap identified in PR #78913, which established that portaled overlay surfaces should map to `md` radius. Reviews confirmed alignment with existing native tooltip behavior and the broader Design System tokens. Merged at commit `cca80fe` with a +35 B size footprint; reported flaky e2e tests were unrelated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

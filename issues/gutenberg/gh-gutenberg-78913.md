# #78913: Add xl border radius token for page shell surfaces.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Package] Edit Site`, `[Package] Theme`
- **Merged:** [`8be2bc3`](https://github.com/WordPress/gutenberg/commit/8be2bc38144900ef7f7f1a99970b0d02a2123008)
- **Discussion:** [#78913](https://github.com/WordPress/gutenberg/pull/78913) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds a new `--wpds-border-radius-xl` (12px) token to the WordPress Design System border radius scale and applies it to outer page shell surfaces in the block editor and boot layout. This resolves a visual hierarchy issue where nested cards and notices previously shared the same 8px radius as the parent shell, making them appear flat. The token is now available for theme and plugin developers to use for large container surfaces that need to visually nest smaller components.

## Impact

- **Theme & plugin developers:** Can now use `--wpds-border-radius-xl` for large container surfaces that require a distinct visual hierarchy from nested components.
- **Site editors & block theme authors:** The site editor layout and boot layout chrome now render with 12px outer corners instead of 8px, improving visual depth in non-full-canvas modes.
- **No breaking changes:** Existing `--wpds-border-radius-lg` usage on cards, notices, and dialogs remains unchanged. No migration or configuration changes are required.

## Technical details

The diff introduces `--wpds-border-radius-xl: 12px` to the WPDS token scale in `packages/theme/tokens/border.json` and regenerates the prebuilt artifacts (`design-tokens.css`, `design-token-fallbacks.mjs`, `design-tokens.mjs`, `token-types.ts`). The TypeScript type `BorderRadiusSize` is extended to include `'xl'`.

In `packages/boot/src/components/root/style.scss`, hardcoded `border-radius: 8px` on `.boot-layout__stage`, `.boot-layout__inspector`, and `.boot-layout__canvas` is replaced with the new token:

```css
/* Before */
border-radius: 8px;

/* After */
border-radius: var(--wpds-border-radius-xl);
```

Similar replacements occur in `packages/boot/src/view-transitions.scss` for `::view-transition-new(boot--inspector)` and `::view-transition-old(boot--inspector)`, and in `packages/edit-site/src/components/layout/style.scss` for `.edit-site-layout__area` and the non-full-canvas preview frame. Nested surfaces like `@wordpress/ui` Card and Notice intentionally retain `--wpds-border-radius-lg` (8px).

## Contribution

During review, a contributor questioned whether the outer shell radius should be so pronounced, noting potential future scale shifts that might make 12px appear too bubbly. The author clarified that token values are independent of future scale iterations and that the `xl` step addresses an immediate design language gap; the team agreed to merge the change and iterate on per-scale values later.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

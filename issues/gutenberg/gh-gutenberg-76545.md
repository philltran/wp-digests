# #76545: Theme package: Add element size design tokens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `Design System`, `[Package] Theme`
- **Merged:** [`81c4b2f`](https://github.com/WordPress/gutenberg/commit/81c4b2f2a95b76161d4ab05eb51025f297f97889)
- **Discussion:** [#76545](https://github.com/WordPress/gutenberg/pull/76545) · 34 comments · 2 reactions

## Summary

The `@wordpress/theme` package introduces a new `size` category to its dimension token family, providing eight standardized CSS custom properties for element sizing ranging from 4px to 40px. This change replaces hardcoded pixel values in interactive components with a shared, base-8 spacing-aligned scale to improve visual consistency across the design system.

## Impact

- **Theme & Design System Developers**: The new `--wpds-dimension-size-*` variables are now available for direct consumption. No APIs were removed or deprecated.
- **Plugin/Theme Developers**: Updating to this release makes the tokens immediately available in prebuilt CSS, JS token lists, and TypeScript definitions. Existing hardcoded sizes remain functional but will not be automatically migrated by core.
- **Required Actions**: None required for typical site operations. Developers using the design system should audit component styles and migrate hardcoded pixel values to the new `--wpds-dimension-size-*` variables where appropriate.

## Technical details

The implementation modifies `packages/theme/tokens/dimension.json` to add a `size` group mapping eight token names (`5xs` through `lg`) to existing space primitives (`wpds-dimension.primitive.space.10` through `space.80`). A corresponding `ElementSize` pattern is registered in `terrazzo.config.ts` to process the new scale. Prebuilt outputs are regenerated across:
- `packages/theme/src/prebuilt/css/design-tokens.css`: Exports variables like `--wpds-dimension-size-lg: 40px;`.
- `packages/theme/src/prebuilt/js/design-tokens.mjs` & `design-token-fallbacks.mjs`: Add token keys and fallback values to the exported lists.
- `packages/theme/src/prebuilt/ts/token-types.ts`: Introduces a strict union type `export type ElementSize = '5xs' | '4xs' | '3xs' | '2xs' | 'xs' | 'sm' | 'md' | 'lg';`.
The diff confirms the build pipeline successfully wires the new JSON primitives into CSS, JS, and TypeScript consumers without altering existing gap or padding schemas.

## Contribution

Opened by @jameskoster and merged after review co-authored by @mirka, @ciampo, @aduth, @jasmussen, and @simison. The discussion focused on whether to include intermediate sizes like 20px and 36px; the team agreed to stick to a base-8 scale and explicitly avoided 36px. An `xl` token for 48px tab heights was discussed but deferred to focus on establishing the core set first. The final merged diff reflects the approved base-8 size scale.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #79773: Theme: Restrict seed colors to opaque values

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`c21e5ac`](https://github.com/WordPress/gutenberg/commit/c21e5ac588225db9233b5da82cd6ee515d3382cd)
- **Discussion:** [#79773](https://github.com/WordPress/gutenberg/pull/79773) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `ThemeProvider` component now strictly rejects partially transparent and `transparent` seed colors for `color.primary` and `color.background`. This change enforces fully opaque sRGB values to align with the stable runtime contract and prevent undefined behavior during color ramp generation. Developers passing alpha values or the `transparent` keyword to these props will now encounter a validation error instead of silent or inconsistent processing.

## Impact

- **Plugin & theme developers:** If you pass `color.primary` or `color.background` to `ThemeProvider` with alpha values (e.g., `rgba(56,88,233,0.3)`, `#3858e94d`, or `transparent`), the component will now throw an error. Update these props to use fully opaque values (e.g., `rgb(56,88,233,1)` or `#3858e9ff`).
- **Headless & REST consumers / Platform teams:** No direct impact unless you rely on the `@wordpress/theme` package for theme generation.
- **No action required** if you only use opaque hex, `rgb()`, `rgba()` (with alpha=1), or CSS named colors.

## Technical details

The change modifies `packages/theme/src/color-ramps/lib/color-utils.ts`, specifically the `assertValidSeedColor` function. Previously, it only validated that the input was an sRGB-parseable string. The updated implementation now destructures the `alpha` property from the parsed color object and throws an error if `alpha !== 1`.

```ts
// Before
let spaceId: string;
try {
  ( { spaceId } = parse( seed ) );
} catch {
  throw new Error( `Unsupported seed color "${ seed }": expected a hex value, an \`rgb()\`/\`rgba()\` string, or a CSS named color.` );
}
if ( ! ALLOWED_SEED_COLOR_SPACES.some( ( space ) => space.id === spaceId ) ) {
  throw new Error( `Unsupported seed color "${ seed }": expected a hex value, an \`rgb()\`/\`rgba()\` string, or a CSS named color, but received a \`${ spaceId }\` color.` );
}

// After
let parsedColor: ReturnType< typeof parse >;
try {
  parsedColor = parse( seed );
} catch {
  throw new Error( `Unsupported seed color "${ seed }": expected a fully opaque hex value, an \`rgb()\`/\`rgba()\` string, or a CSS named color.` );
}
const { alpha = 1, spaceId } = parsedColor;
if ( ! ALLOWED_SEED_COLOR_SPACES.some( ( space ) => space.id === spaceId ) ) {
  throw new Error( `Unsupported seed color "${ seed }": expected a fully opaque hex value, an \`rgb()\`/\`rgba()\` string, or a CSS named color, but received a \`${ spaceId }\` color.` );
}
if ( alpha !== 1 ) {
  throw new Error( `Unsupported seed color "${ seed }": expected a fully opaque color.` );
}
```

The TypeScript interfaces in `packages/theme/src/types.ts` and the README are updated to document that `color.primary` and `color.background` require fully opaque values. Unit tests in `packages/theme/src/color-ramps/test/seed-input.test.ts` move previously accepted transparent values to the `REJECTED_SEEDS` array and update accepted values to explicitly use alpha=1 or `ff` suffixes.

## Contribution

Opened and merged by @ciampo, with review contributions from @mirka. The PR was created as a follow-up to issue #77462 to enforce the stable runtime contract regarding alpha channels in seed colors. The implementation focused on adding an alpha check to the existing validator, updating documentation, and adjusting tests. No major design alternatives were debated; the change directly addresses the stated contract requirement.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #78741: Theme: drop `density` support from `@wordpress/theme`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`f461948`](https://github.com/WordPress/gutenberg/commit/f4619487fdfb9a1a48faa6afe5c8c97c9af2a6da)
- **Discussion:** [#78741](https://github.com/WordPress/gutenberg/pull/78741) · 2 comments · 0 reactions

## Summary

`@wordpress/theme`'s `ThemeProvider` component has dropped its experimental `density` prop and all supporting infrastructure. The feature was flagged during the `ThemeProvider` stabilization review (#77462) as having unresolved propagation problems, asymmetric prop semantics, and no agreed-upon design direction. Default dimension token values (`--wpds-dimension-padding-*`, `--wpds-dimension-gap-*`) continue to be emitted at `:root` exactly as before, so the change is invisible to anyone not using the prop. Density support is explicitly deferred — not abandoned — pending a stable design.

## Impact

**Plugin & theme developers / package consumers**
- **Breaking change:** The `density` prop has been removed from `ThemeProvider` and from the `ThemeProviderSettings` TypeScript interface. Passing `density` now produces a TypeScript compile error; at runtime the component falls back to default spacing.
- The `data-wpds-density` DOM attribute is no longer written to the `ThemeProvider` wrapper element. Any CSS that targeted `[data-wpds-density="compact"]` or `[data-wpds-density="comfortable"]` will silently stop applying.
- The per-density override blocks in the prebuilt `design-tokens.css` (and the generated `tokens/modes/dimension.compact.json` / `tokens/modes/dimension.comfortable.json` files) are gone. Downstream build pipelines that consumed those files directly must remove the references.
- A repo-wide grep found **no usage of the `density` prop outside the package's own stories**, so real-world breakage is expected to be negligible.

**Consumers not passing `density`**
- No action required. Visual output and token values are unchanged.

## Technical details

**Removed from `ThemeProvider`:**
- The `density` prop and its union type (`'compact' | 'comfortable' | 'default'`) from `ThemeProviderSettings` in `packages/theme/src/`.
- The `data-wpds-density` attribute that the component wrote to its wrapper `<div>`.

**Removed from the token layer:**
- `compact` and `comfortable` override entries for `--wpds-dimension-padding-*` and `--wpds-dimension-gap-*` in the source token JSON.
- Generated output files `packages/theme/tokens/modes/dimension.compact.json` and `packages/theme/tokens/modes/dimension.comfortable.json`.
- The `compact`, `comfortable`, and `default` density mode selectors from the Terrazzo configuration.
- The corresponding scoped CSS blocks in `packages/theme/design-tokens.css` (the prebuilt stylesheet).

**What remains unchanged:**
- All `--wpds-dimension-padding-*` and `--wpds-dimension-gap-*` default values continue to be emitted under `:root`.

**Usage change:**
```tsx
// Before — accepted without error
<ThemeProvider density="compact">…</ThemeProvider>

// After — TypeScript error; density prop no longer in ThemeProviderSettings
<ThemeProvider>…</ThemeProvider>
```

**Build size impact:** net −168 B across `build/modules/boot/index.min.js` (−155 B) and `build/scripts/theme/index.min.js` (−13 B).

The change is recorded as a **Breaking Change** in `packages/theme/CHANGELOG.md` under the next unreleased version.

## Contribution

Opened and merged by @ciampo (Marco Ciampini) as task **T-X3** of the `ThemeProvider` stabilization review tracked in #77462. @mirka is credited in the merge commit attribution. The PR discussion in the provided material consists solely of automated size-report and props-bot comments — no alternative approaches or design debate are visible. Related prior art on the density design problem lives in #74556.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

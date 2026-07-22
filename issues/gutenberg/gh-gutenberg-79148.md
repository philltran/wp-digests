# #79148: Theme: Enforce sRGB seed-color input contract for ThemeProvider

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`c4982f1`](https://github.com/WordPress/gutenberg/commit/c4982f137915d53306d7b6e7cf0ea2585c56bd74)
- **Discussion:** [#79148](https://github.com/WordPress/gutenberg/pull/79148) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `ThemeProvider` component in the `@wordpress/theme` package now strictly enforces its documented input contract for `color.primary` and `color.background` seed colors. Previously, non-sRGB color strings (such as `oklch()` or `hsl()`) were accidentally accepted due to internal `colorjs.io` registration side effects. The change adds deterministic validation at the `buildRamp` entry point, throwing a clear error for any color space outside sRGB (hex, `rgb()`/`rgba()`, or CSS named colors).

## Impact

- **Plugin & theme developers:** Passing non-sRGB color strings to `ThemeProvider`'s `color.primary` or `color.background` props will now throw a runtime `Error` instead of silently accepting them.
- **Breaking change:** The implicit acceptance of `oklch()`, `hsl()`, `lab()`, `hwb()`, and `color(display-p3)` seed strings is removed. Replace these with sRGB-compatible equivalents (`#hex`, `rgb()`, `rgba()`, or CSS named colors) before passing them to `ThemeProvider`.
- **No action required** for developers already using hex, `rgb()`/`rgba()`, or CSS named colors.

## Technical details

The diff introduces `assertValidSeedColor()` in `packages/theme/src/color-ramps/lib/color-utils.ts`, which registers the `sRGB` space, parses the seed string via `colorjs.io/fn`'s `parse()`, and validates that the resulting `spaceId` matches the `ALLOWED_SEED_COLOR_SPACES` allowlist (`[ sRGB ]`). This validation is invoked at the single user-input chokepoint, `buildRamp()`, in `packages/theme/src/color-ramps/lib/index.ts`, ensuring internal recursive callers (which pass `PlainColorObject` instances) bypass string validation. The function throws a descriptive `Error` for invalid inputs.

**Before (silently accepted):**
```js
<ThemeProvider color={{ primary: 'oklch(0.7 0.15 250)' }} />
```

**After (throws):**
```js
<ThemeProvider color={{ primary: 'oklch(0.7 0.15 250)' }} />
// Error: Unsupported seed color "oklch(0.7 0.15 250)": expected a hex value, an `rgb()`/`rgba()` string, or a CSS named color, but received a `oklch` color.
```

Type definitions in `packages/theme/src/types.ts` and documentation in `README.md` and `CHANGELOG.md` are updated to reflect the strict sRGB-only contract.

## Contribution

Opened and merged by @ciampo with review from @aduth. During review, @aduth questioned whether the explicit allowlist would remain necessary if the upstream `colorjs.io` fix for `OKLCH` registration lands. @ciampo demonstrated that because `ColorSpace.registry` is a process-wide singleton, relying on default parsing behavior would make acceptance non-deterministic across different `colorjs.io` consumers. The explicit allowlist was retained to guarantee deterministic sRGB-only validation regardless of global registry state.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

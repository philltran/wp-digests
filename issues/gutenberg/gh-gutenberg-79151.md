# #79151: Theme: differentiate `--wpds-color-fg-interactive-{brand,error}-active` vs resting state tokens

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`a7230ec`](https://github.com/WordPress/gutenberg/commit/a7230ec97f149f2512665850cd421e853ced1f1d)
- **Discussion:** [#79151](https://github.com/WordPress/gutenberg/pull/79151) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The WordPress Design System (WPDS) theme package now assigns distinct CSS custom property values to the `--wpds-color-foreground-interactive-brand-active` and `--wpds-color-foreground-interactive-error-active` tokens, differentiating them from their resting states. Previously, hover, focus, and active states shared the exact same color as the default state, reducing visual discernibility. This change improves accessibility and UI feedback for interactive elements without altering token names or structure.

## Impact

- Theme and block developers using the WPDS theme package will see updated hover/focus colors for brand and error interactive elements.
- No breaking changes or migration steps required; existing token names remain unchanged.
- Site owners and end users benefit from clearer interactive state feedback, though no direct configuration is needed.

## Technical details

The diff remaps the `brand-active` and `error-active` tokens from `fgSurface3` to `fgSurface4` in `packages/theme/tokens/color.json`. This cascades into `packages/theme/src/prebuilt/css/design-tokens.css`, where `--wpds-color-foreground-interactive-brand-active` shifts from `#3858e9` to `#0b0070` and `--wpds-color-foreground-interactive-error-active` shifts from `#cc1818` to `#470000`. The fallback mapping in `packages/theme/src/prebuilt/js/design-token-fallbacks.mjs` is updated to use `color-mix(in oklch, var(--wp-admin-theme-color, #3858e9) 52%, black)` for the brand active state. Corresponding TypeScript mappings in `packages/theme/src/prebuilt/ts/color-tokens.ts` are adjusted to reflect the new `fgSurface4` assignments, ensuring the design system's internal primitive references align with the updated visual states.

## Contribution

Opened by @ciampo to address visual state ambiguity noted in #77153, the author collaborated with @jameskoster during a pairing session to refine the token mapping. The approach focused solely on remapping existing primitives rather than introducing new tokens or altering the design system architecture. @mirka co-authored and merged the PR after a brief review cycle, with the change landing as a straightforward enhancement to the theme package.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

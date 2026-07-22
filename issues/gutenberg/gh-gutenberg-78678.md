# #78678: Theme: apply ThemeProvider styles inline (I2)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Theme`
- **Merged:** [`12358ae`](https://github.com/WordPress/gutenberg/commit/12358ae0038c4d11fcea8e16d91786f223a8cda4)
- **Discussion:** [#78678](https://github.com/WordPress/gutenberg/pull/78678) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `ThemeProvider` component in the `@wordpress/theme` package now applies dynamic theme tokens (colors, cursors) as inline custom properties on its wrapper element instead of injecting a per-instance `<style>` tag. When `isRoot` is true, these properties are mirrored onto the document element via a layout effect, ensuring they propagate correctly to portals and iframes. This change reduces DOM node count, eliminates a runtime specificity hack, and simplifies the scoping model without altering the public component API.

## Impact

- **Plugin & theme developers:** No immediate code changes required. The public `<ThemeProvider>` interface remains unchanged.
- **Custom DOM queries:** Selectors targeting `[data-wpds-theme-provider-id]` or parsing the injected `<style>` element will break; update to query the wrapper element and read inline `style` properties instead.
- **Platform & hosting:** Slightly lighter DOM output per provider instance and reduced CSS specificity conflicts.
- **No action required** for standard usage of the theme package or preset-based token forwarding.

## Technical details

- Replaces the `<style>` injection and `useId`-based scoping in `packages/theme/src/theme-provider.tsx` with direct `style={ themeProviderStyles }` on the wrapper `<div>`.
- Introduces a `useIsomorphicLayoutEffect` hook (imported from `@wordpress/compose`) that, when `isRoot` is true, iterates over `themeProviderStyles`, sets each custom property on `wrapperRef.current.ownerDocument.documentElement`, and restores previous values on unmount.
- Drops `data-wpds-theme-provider-id`, the doubled-class specificity hack, and the runtime `:root:has(…)` selector for dynamic tokens.
- Preserves `data-wpds-root-provider="true"` and `data-wpds-corner-radius` for preset-based values, which continue to be forwarded via the prebuilt CSS `:root:has( [data-wpds-root-provider='true']… )` rule.
- Adds a dev-only warning when multiple `isRoot` providers mount on the same document, tracked via a `WeakMap<Document, number>`.
- Fixes an iframe rendering bug by using `ownerDocument` instead of the global `document` when mirroring styles.
- Adds `@wordpress/compose` as a dependency in `packages/theme/package.json` and updates `tsconfig` references to include `../compose`.

## Contribution

The PR was authored by @ciampo and refined during review with @mirka. The approach evolved to address iframe rendering by switching from `document` to `ownerDocument`, and a runtime warning for conflicting root providers was added mid-review. The design settled on an asymmetry between dynamic token forwarding (JS inline styles) and preset-based forwarding (static CSS), resolving the scoping complexity discussed in the parent issue without introducing a new public API.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #78816: Add corner radius presets to ThemeProvider

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `Design System`, `[Package] Theme`
- **Merged:** [`ef0f9e2`](https://github.com/WordPress/gutenberg/commit/ef0f9e2d3419a7066e7caf84b69dfe61474b313f)
- **Discussion:** [#78816](https://github.com/WordPress/gutenberg/pull/78816) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds a `cornerRadius` prop to `@wordpress/theme`'s `ThemeProvider`, allowing developers to apply a global border-radius preset (`none`, `subtle`, `moderate`, or `pronounced`) to a React subtree. The change replaces the previously static border-radius scale with four prebuilt design-token modes, enabling consistent UI roundness without manual CSS overrides.

## Impact

- **Plugin & theme developers using `@wordpress/theme`/`@wordpress/ui`:** Can now pass `cornerRadius` to `ThemeProvider` to globally adjust component roundness across a subtree.
- **Design system consumers:** No action required; the default `'subtle'` preset preserves the existing static radius values.
- **Per-component overrides:** Not supported in this release; the preset applies uniformly across the `--wpds-border-radius-*` scale rather than targeting individual components.

## Technical details

The diff introduces a `cornerRadius` prop to `ThemeProvider` (`packages/theme/src/theme-provider.tsx`) that resolves via React context inheritance, defaulting to `'subtle'`. The provider renders `data-wpds-corner-radius={cornerRadiusPreset}` on its root element. Build-time Terrazzo token modes (`packages/theme/tokens/border.json` and `packages/theme/terrazzo.config.ts`) generate prebuilt CSS in `design-tokens.css` that overrides `--wpds-border-radius-xs` through `--wpds-border-radius-xl` based on the attribute selector. The `CornerRadiusPreset` type is exported from `@wordpress/theme`. Runtime CSS injection is avoided; values are baked into the token pipeline.

**Before:**
```jsx
<ThemeProvider color={ themeColors }>
  <App />
</ThemeProvider>
```

**After:**
```jsx
<ThemeProvider color={ themeColors } cornerRadius="moderate">
  <App />
</ThemeProvider>
```

## Contribution

Opened and merged by @jameskoster with co-authors @mirka, @aduth, @ciampo, and @jasmussen. Early iterations explored six presets and uniform multipliers, but feedback led to reducing the scale to four hand-tuned presets that compress toward a shared radius at larger sizes. Per-component radius overrides were discussed but deferred to a future iteration to keep the initial scope focused on global theme inheritance.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

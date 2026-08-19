# #81653: Site Editor: Use inverted ThemeProvider seed for portaled UI

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Package] Edit Site`
- **Merged:** [`75b9133`](https://github.com/WordPress/gutenberg/commit/75b91332226b3e097abd99d4de1c0fce09141ef9)
- **Discussion:** [#81653](https://github.com/WordPress/gutenberg/pull/81653) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Editor now wraps its layout in an outer root `ThemeProvider` that seeds the admin color scheme's `primary` color and a white content background, while the existing `ThemeProvider` remains nested for the editor chrome. This fixes portaled popovers and modals that render outside the original `ThemeProvider` wrapper and therefore missed the chosen admin scheme's accent colors. It backports the inverted `ThemeProvider` seed pattern from #81296 to trunk. The change is a broad stopgap until portaled components are fully theme-ready.

## Impact

- **Site Editor users:** Portaled UI such as popovers and modals should now reflect the selected admin color scheme's primary/accent colors, while the Site Editor chrome keeps the scheme background.
- **Plugin and theme developers:** No public API, hook, REST route, or configuration change. If you build WPDS-styled controls inside Site Editor popovers or modals, they may now receive the expected accent colors.
- **Hosting and platform teams:** No migration or configuration required.
- **Headless and REST consumers:** No impact.

## Technical details

In `packages/edit-site/src/components/layout/index.js`, `LayoutWithGlobalStylesProvider` now nests two `ThemeProvider` components. The outer provider uses `isRoot` and seeds only the admin scheme's `primary` color plus `CONTENT_COLOR`, so portaled content outside the editor wrapper can pick up the accent and a white content background. The inner provider keeps the full `themeColors` for the Site Editor chrome.

Before:

```jsx
<ThemeProvider color={ themeColors }>
  <Layout { ...props } />
</ThemeProvider>
```

After:

```jsx
<ThemeProvider
  isRoot
  color={ {
    primary: themeColors.primary,
    ...CONTENT_COLOR,
  } }
>
  <ThemeProvider color={ themeColors }>
    <Layout { ...props } />
  </ThemeProvider>
</ThemeProvider>
```

The fix addresses DOM portaling: popovers and modals render outside the ancestor `ThemeProvider` wrapper, so they do not inherit CSS custom properties set on that wrapper.

## Contribution

The PR backports an inverted `ThemeProvider` layout previously merged to `wp/7.1`. In review, @ciampo suggested an end-to-end regression test for the color split and tracking cleanup for removing the workaround once components are theme-ready. The author declined the test as low-value and cosmetic, said cleanup tracking was not currently prioritizable, and raised a design question about whether popovers triggered from a dark sidebar should inherit dark mode.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #78397: Site Editor: Apply the user's admin color scheme

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`, `[Package] Edit Site`, `[Package] Base styles`
- **Merged:** [`d18be22`](https://github.com/WordPress/gutenberg/commit/d18be229f2c87df5d3d5ee920d1c7b0d8a68afea)
- **Discussion:** [#78397](https://github.com/WordPress/gutenberg/pull/78397) · 36 comments · 0 reactions

## Summary

The Site Editor UI chrome now dynamically applies the active WordPress admin color scheme instead of rendering a fixed dark background. This enhancement extracts theme-color logic into `@wordpress/admin-ui`, enabling both the boot layout and site editor to share a consistent design system seed derived from the user’s profile settings.

## Impact

- **Site editors & dashboard users**: Sidebar and shell backgrounds now match the selected admin color scheme (e.g., Light, Coffee, Blue).
- **Plugin/theme developers**: `@wordpress/boot` no longer uses a self-contained `UserThemeProvider`; theme coloring is centralized in `@wordpress/admin-ui` and consumed via `@wordpress/theme`'s `ThemeProvider`. Existing usage patterns remain functionally equivalent but are internally refactored.
- **Site Editor v2**: Excluded from this change until the admin color class is properly injected into the `<body>` element; note the limitation if testing canvas/edit modes in v2.
- **No migration required** for end users or consumers relying on standard editor interfaces.

## Technical details

- Introduces `getAdminThemeColors()` in `packages/admin-ui/src/admin-theme-colors/index.ts`, which parses the `admin-color-[\w-]+` class from `document.body.className` and returns a mapped `{ primary, bg }` object for each registered scheme.
- Replaces the localized `UserThemeProvider` in `packages/boot/src/components/root/index.tsx` and `single-page.tsx` with `@wordpress/theme`'s `ThemeProvider`, passing extracted colors as initial seeds (`color={ { ...themeColors, bg: '#f8f8f8' } }`).
- Updates `packages/edit-site/src/components/layout/index.js` to wire `ThemeProvider` around layout regions. Mobile viewports are explicitly split: `areas.mobileSidebar` receives the themed seed, while `areas.mobileContent` and `preview` are wrapped with a fixed white content background (`CONTENT_COLOR = { bg: '#ffffff' }`).
- Syncs SCSS mixin values in `packages/base-styles/_mixins.scss` to match the new primary/background hex codes, ensuring CSS-generated admin scheme backgrounds align with the JS-driven theme provider.

## Contribution

Merged by @fushar following a review cycle involving @ciampo, @jameskoster, @youknowriad, and others. The initial approach was refined to use proper WPDS tokens rather than SCSS overrides, with reviewers recommending seed value adjustments to improve contrast on color ramps. Site Editor v2 support was deferred after identifying that the admin color class is not yet applied to the v2 body element. Co-authored by multiple core contributors during the review process.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

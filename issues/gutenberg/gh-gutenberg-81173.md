# #81173: Widgets editor: Add ThemeProvider for admin color schemes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Edit Widgets`, `Backported to WP Core`
- **Merged:** [`b8cb89c`](https://github.com/WordPress/gutenberg/commit/b8cb89c907f838dd1cc39a6f7e1f1d8195da3385)
- **Discussion:** [#81173](https://github.com/WordPress/gutenberg/pull/81173) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The widgets editor now wraps its layout in a `ThemeProvider` seeded with the active admin color scheme’s primary color. This aligns the widgets editor with the post and site editors, ensuring WPDS-styled UI components and portaled elements correctly inherit the user’s chosen admin color scheme instead of falling back to default design tokens.

## Impact

- **Plugin & theme developers:** No direct code changes required. Custom widgets editor UI or portaled components will now automatically inherit the admin color scheme’s primary color.
- **Site owners & editors:** The widgets editor interface will visually match the selected admin color scheme (e.g., Ocean, Midnight, Sunrise) for primary accents, improving consistency across the admin.
- **Hosting & platform teams:** No configuration changes needed. The change is self-contained within the `@wordpress/edit-widgets` package.
- **No action required** for existing plugins, themes, or workflows.

## Technical details

The diff modifies `packages/edit-widgets/src/components/layout/index.js` to import `getAdminThemeColors` from `@wordpress/admin-ui` and `ThemeProvider` from `@wordpress/theme`. It computes the active scheme’s primary color via `useMemo` and wraps the entire `Layout` component tree in `<ThemeProvider isRoot color={ { primary: adminPrimary } }>`. The `isRoot` prop ensures tokens are forwarded to the document element, allowing portaled UI to inherit them. Background tokens intentionally fall back to the design system default (`#fcfcfc`) since the widgets screen uses white surfaces. The `@wordpress/admin-ui` and `@wordpress/theme` packages are added as dependencies to `packages/edit-widgets/package.json`.

**Before:**
```jsx
<ErrorBoundary>
  <div { ...navigateRegionsProps }>
    <WidgetAreasBlockEditorProvider blockEditorSettings={ blockEditorSettings }>
      <Interface blockEditorSettings={ blockEditorSettings } />
      <Sidebar />
      <PluginArea onError={ onPluginAreaError } />
      <UnsavedChangesWarning />
      <WelcomeGuide />
    </WidgetAreasBlockEditorProvider>
  </div>
</ErrorBoundary>
```

**After:**
```jsx
<ThemeProvider isRoot color={ { primary: adminPrimary } }>
  <ErrorBoundary>
    <div { ...navigateRegionsProps }>
      <WidgetAreasBlockEditorProvider blockEditorSettings={ blockEditorSettings }>
        <Interface blockEditorSettings={ blockEditorSettings } />
        <Sidebar />
        <PluginArea onError={ onPluginAreaError } />
        <UnsavedChangesWarning />
        <WelcomeGuide />
      </WidgetAreasBlockEditorProvider>
    </div>
  </ErrorBoundary>
</ThemeProvider>
```

## Contribution

Opened by @mirka as a direct follow-up to #81112, the change was reviewed and merged with co-authors @t-hamano and @aduth. The PR encountered a merge conflict during the automated cherry-pick to `wp/7.1`, which was manually resolved before final inclusion. The record carries no significant design debate or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #81112: Post editor: Add ThemeProvider for admin color schemes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Package] Edit Post`, `Backported to WP Core`
- **Merged:** [`7065076`](https://github.com/WordPress/gutenberg/commit/7065076923bf1bcdb362ee66663f5f2d0f735be3)
- **Discussion:** [#81112](https://github.com/WordPress/gutenberg/pull/81112) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The post editor layout is now wrapped in a `ThemeProvider` that injects the active admin color scheme’s primary color as a design token. This resolves a 7.1 beta regression where WPDS components in the editor (sidebar panels, tooltips, meta box handles, and portaled modals) failed to reflect the selected admin color scheme. The change aligns the post editor’s theming behavior with the site editor, which already uses a root `ThemeProvider`.

## Impact

- **Plugin & theme developers:** No breaking changes or migration steps required. Existing editor extensions continue to render, but portaled UI components and WPDS-based elements will now correctly inherit the active admin color scheme’s primary token.
- **Hosting & platform teams:** No configuration changes needed. The change is self-contained within the `@wordpress/edit-post` package.
- **General developers:** No action required. The fix applies automatically upon upgrading to the version containing this PR.

## Technical details

The diff modifies `packages/edit-post/src/components/layout/index.js` to import `getAdminThemeColors` from `@wordpress/admin-ui` and `ThemeProvider` from the newly added `@wordpress/theme` dependency. Inside the `Layout` component, `adminPrimary` is memoized via `useMemo( () => getAdminThemeColors().primary, [] )`. The editor shell is then wrapped in `<ThemeProvider isRoot color={ { primary: adminPrimary } }>` immediately after `SlotFillProvider` and `Tooltip.Provider`. Background tokens intentionally fall back to the design system default (`#fcfcfc`) since the post editor uses white surfaces. The `isRoot` prop forwards the token to the document element, ensuring portaled UI inherits the color correctly.

```jsx
// Before
<SlotFillProvider>
  <Tooltip.Provider>
    <ErrorBoundary canCopyContent>
      {/* editor shell */}
    </ErrorBoundary>
  </Tooltip.Provider>
</SlotFillProvider>

// After
<SlotFillProvider>
  <Tooltip.Provider>
    <ThemeProvider isRoot color={ { primary: adminPrimary } }>
      <ErrorBoundary canCopyContent>
        {/* editor shell */}
      </ErrorBoundary>
    </ThemeProvider>
  </Tooltip.Provider>
</SlotFillProvider>
```

## Contribution

Identified as a regression in the 7.1 beta cycle, the PR aligned the post editor’s theming layer with the site editor’s existing pattern. The author proposed wrapping the layout in a root `ThemeProvider` seeded with the active admin color scheme, which reviewers accepted as a minimal, self-contained fix. The change was merged and subsequently backported to the release branch to address the beta regression before the stable launch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

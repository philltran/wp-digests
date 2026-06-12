# #79056: Site Editor: Fix admin color scheme bleeding through the mobile content scrollbar gutter

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Bug`, `[Package] Edit Site`
- **Merged:** [`6c76206`](https://github.com/WordPress/gutenberg/commit/6c762064e003e1281ac4555a3af620796658fd53)
- **Discussion:** [#79056](https://github.com/WordPress/gutenberg/pull/79056) · 4 comments · 0 reactions

## Summary

Fixes a visual regression in the Site Editor mobile layout where the admin color scheme background bled through the scrollbar gutter behind the content panel. The change moves the `<ThemeProvider>` wrapper to encompass the scroll container, ensuring the gutter inherits the neutral surface token and renders a clean white edge.

## Impact

['- **Site Editors/Developers**: No action required. Resolves a mobile UI artifact in the Style Editor layout where theme colors appeared behind the white content panel.', '- **Theme/Plugin Developers**: Minimal impact. Modifies internal `.edit-site-sidebar__screen-wrapper` styling and JavaScript component hierarchy; no public APIs or external modules are affected.']

## Technical details

['**File**: `packages/edit-site/src/components/layout/index.js`', '**Behavior Change**: In the `Layout` component, the rendering order for `areas.mobileContent` is restructured. Previously, `<ThemeProvider color={ CONTENT_COLOR }>` wrapped only the inner content div inside `<SidebarContent>`. The diff moves `<ThemeProvider>` to wrap `<SidebarContent>` directly.', '```jsx\n// Before\n<SidebarContent routeKey={ routeKey }>\n  { areas.mobileContent ? (\n    <ThemeProvider color={ CONTENT_COLOR }>\n      {/* Content */}\n    </ThemeProvider>\n  ) : null }\n</SidebarContent>\n\n// After\n{ areas.mobileContent ? (\n  <ThemeProvider color={ CONTENT_COLOR }>\n    <SidebarContent routeKey={ routeKey }>\n      {/* Content */}\n    </SidebarContent>\n  </ThemeProvider>\n) : null }\n```', '**CSS Update**: `packages/edit-site/src/components/layout/style.scss` adds a rule targeting the scroll wrapper when mobile content is present:', '```css\n.edit-site-sidebar__screen-wrapper:has(.edit-site-layout__mobile-content) {\n  background: var(--wpds-color-bg-surface-neutral);\n}\n```']

## Contribution

['PR #79056 by @fushar, merged by @ciampo. The fix addresses a previous discussion on scrollbar rendering in mobile layouts by restructuring the component tree to apply theme context to the wrapper rather than just the inner content block.']

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

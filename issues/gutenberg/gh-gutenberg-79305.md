# #79305: Site Editor: Handle `aria-current` natively in `SidebarNavigationItem`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Edit Site`, `[Feature] Site Editor`
- **Merged:** [`d1e67aa`](https://github.com/WordPress/gutenberg/commit/d1e67aa5c8128d510924561708e8f7ad149bf6e3)
- **Discussion:** [#79305](https://github.com/WordPress/gutenberg/pull/79305) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Editor’s `SidebarNavigationItem` component now natively handles active-state marking via a new `activeOnRouteName` prop, eliminating two thin wrapper components (`SidebarNavigationItemGlobalStyles` and `SidebarNavigationItemIdentity`). This centralizes route-based `aria-current` logic directly in the navigation item, simplifying the sidebar’s component tree and reducing abstraction overhead.

## Impact

- **Plugin & theme developers:** No direct impact; the removed wrappers were internal to `@wordpress/edit-site`.
- **Site Editor users:** No behavioral change; sidebar navigation continues to highlight the active route identically.
- **Internal consumers:** Any custom code directly importing `SidebarNavigationItemGlobalStyles` or `SidebarNavigationItemIdentity` will break. Replace them with `SidebarNavigationItem` and pass `activeOnRouteName="styles"` or `activeOnRouteName="identity"` as needed.

## Technical details

The diff modifies `packages/edit-site/src/components/sidebar-navigation-item/index.js` to import `useLocation` from `@wordpress/router` and accept an `activeOnRouteName` prop. When provided, it computes `isActive = activeOnRouteName && name === activeOnRouteName` and applies `aria-current={ isActive ? true : undefined }` to the underlying `Item` component. The two wrapper components in `sidebar-navigation-screen-global-styles/index.js` and `sidebar-navigation-screen-identity/index.js` are deleted. In `sidebar-navigation-screen-main/index.js`, the root items now render `SidebarNavigationItem` directly with the new prop:

**Before:**
```jsx
<SidebarNavigationItemGlobalStyles to="/styles" uid="global-styles-navigation-item" icon={ styles }>
  { __( 'Styles' ) }
</SidebarNavigationItemGlobalStyles>
```

**After:**
```jsx
<SidebarNavigationItem to="/styles" uid="global-styles-navigation-item" icon={ styles } activeOnRouteName="styles">
  { __( 'Styles' ) }
</SidebarNavigationItem>
```

Additionally, `HStack` is replaced with `Stack` from `@wordpress/ui`, and an eslint suppression for `@wordpress/use-recommended-components` is removed.

## Contribution

Merged by @Mamaduka with co-authorship from @ramonJD, the change emerged from a design debate over how to track active routes without extra abstraction. The author considered auto-deriving active state from the `to` prop by parsing `location.path`, but rejected it in favor of the explicit `activeOnRouteName` prop to avoid path-matching complexity. The change was reviewed, passed CI, and merged with a minor size increase (+1.21 kB).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

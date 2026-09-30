# #82624: Site Editor: Restore Global Styles access on the front page

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Package] Editor`, `[Package] Edit Site`
- **Merged:** [`4042508`](https://github.com/WordPress/gutenberg/commit/40425087ad3188980934cee6d3f3126f04826085)
- **Discussion:** [#82624](https://github.com/WordPress/gutenberg/pull/82624) · 17 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

With a static front page configured, clicking **Revisions** in the Site Editor's Styles sidebar opened the page editor in `post-only` rendering mode, which suppressed the Styles sidebar entirely and left the revision history unreachable. The fix defaults the home, identity, and styles routes to `template-locked` rendering mode so the canvas always shows the template (and therefore the Styles sidebar), and reorders the sidebar-open sequence so the `/revisions` path is set before the sidebar mounts and resets its navigation.

## Impact

- **Site owners (block theme, static front page):** Revisions in the Styles sidebar now opens correctly. No configuration change needed.
- **Site owners (block theme, static front page, "Show template" disabled on a page):** The bug persists on the home/identity/styles routes because `getDefaultRenderingMode` checks the saved preference before the new `defaultRenderingMode` prop. This is a documented known limitation.
- **Plugin & theme developers:** No public API change. The `defaultRenderingMode` prop on `EditSiteEditor` and the parameter on `useSpecificEditorSettings` are internal to `@wordpress/edit-site` and not part of the public block-editor API surface.
- **No action required** for the vast majority of sites; the fix is transparent.

## Technical details

Three coordinated changes in `packages/edit-site` and `packages/editor`:

1. **Rendering mode for global routes.** `useSpecificEditorSettings` (in `use-site-editor-settings.js`) gains a `defaultRenderingMode` parameter (default `'post-only'`), forwarded into the editor settings object. `EditSiteEditor` (in `editor/index.jsx`) gains a matching `defaultRenderingMode` prop and passes it through. The three route components — `home.jsx`, `identity.jsx`, and `styles.jsx` — now render `<Editor defaultRenderingMode="template-locked" />` instead of `<Editor />`.

2. **Sidebar open sequence.** In `sidebar-navigation-screen-global-styles/index.jsx` and `use-common-commands.js`, `setStylesPath('/revisions')` is called **before** `openGlobalStyles()` / `openGeneralSidebar('edit-site/global-styles')`. Previously the path was set after the sidebar opened, so the sidebar's mount-time reset discarded it.

3. **Reset guard in the sidebar.** In `global-styles-sidebar/index.jsx`, the `useEffect` that calls `resetStylesNavigation()` on sidebar open now computes `hasRequestedPath = stylesPath !== '/' && ! shouldResetNavigation` and skips the reset when true, preserving the pre-set `/revisions` path.

4. **Styles navigation teardown on exit.** A new `useEffect` in `editor/index.jsx` calls `resetStylesNavigation()` whenever the component is rendered outside the styles route in edit mode (`! isStylesEditing`). Because routes remount this component on navigation, the check runs on every render rather than on a specific transition, covering all exit paths (Back button, canvas click, direct navigation) rather than only the Back button.

```jsx
// Before (home.jsx)
<Editor isHomeRoute />

// After
<Editor isHomeRoute defaultRenderingMode="template-locked" />
```

A new e2e spec in `test/e2e/specs/site-editor/user-global-styles-revisions.spec.js` covers the static-front-page Revisions flow and a follow-up test verifying the site preview canvas remains clickable after leaving the styles editor via the Back button.

## Contribution

Opened by @ramonjd as a follow-up to #72681. @youknowriad pushed back on the initial framing, arguing the real issue was that the home/identity/styles routes rendered in `post-only` mode rather than the Styles sidebar being unreachable, and suggested defaulting those "global routes" to `template-locked` without touching entity resolution. @ramonjd had been weighing two alternatives (teaching `getDefaultRenderingMode` about the front page vs. coaxing the route to resolve as a template) and adopted the route-level default approach. During implementation, the saved "Show template" preference overriding the new prop was identified as a known limitation; the author floated letting an explicit prop outrank the preference but shipped the limitation as-is. Merged as `4042508`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

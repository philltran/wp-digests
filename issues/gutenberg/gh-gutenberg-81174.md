# #81174: Customizer widgets: Add ThemeProvider for admin color schemes

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Feature] Widgets Customizer`, `Backported to WP Core`
- **Merged:** [`1908100`](https://github.com/WordPress/gutenberg/commit/1908100a4755608faf97ebca33c1c283556d3870)
- **Discussion:** [#81174](https://github.com/WordPress/gutenberg/pull/81174) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Customizer widgets editor now wraps its block editor UI in a `ThemeProvider` seeded with the active admin color scheme’s primary color. This aligns the widgets interface with the post and site editors, ensuring design tokens and portaled UI elements inherit the correct admin theme colors. It resolves a visual inconsistency where the widgets editor previously ignored the user’s selected admin color scheme.

## Impact

- **Plugin & theme developers:** No direct code changes required. The modification is internal to the `@wordpress/customize-widgets` package.
- **Site owners & admins:** The Customizer widgets UI will now visually match the active admin color scheme (e.g., Ocean, Midnight, Sunrise), improving contrast on white surfaces.
- **Platform & hosting teams:** No configuration, migration, or upgrade steps required.
- **Action required:** None.

## Technical details

The diff modifies `packages/customize-widgets/src/components/customize-widgets/index.js` to import `getAdminThemeColors` from `@wordpress/admin-ui` and `ThemeProvider` from `@wordpress/theme`. It computes the active primary color via `useMemo( () => getAdminThemeColors().primary, [] )` and wraps the existing `SlotFillProvider` tree with `<ThemeProvider isRoot color={ { primary: adminPrimary } }>`. The `isRoot` prop forwards tokens to the document element so portaled components (like popovers) inherit the theme. Dependencies `@wordpress/admin-ui` and `@wordpress/theme` were added to `packages/customize-widgets/package.json`.

**Before:**
```jsx
<SlotFillProvider>
	<SidebarControls sidebarControls={ sidebarControls } activeSidebarControl={ activeSidebarControl }>
		<FocusControl api={ api } sidebarControls={ sidebarControls }>
			{ activeSidebar }
			{ popover }
		</FocusControl>
	</SidebarControls>
</SlotFillProvider>
```

**After:**
```jsx
<SlotFillProvider>
	<ThemeProvider isRoot color={ { primary: adminPrimary } }>
		<SidebarControls sidebarControls={ sidebarControls } activeSidebarControl={ activeSidebarControl }>
			<FocusControl api={ api } sidebarControls={ sidebarControls }>
				{ activeSidebar }
				{ popover }
			</FocusControl>
		</SidebarControls>
	</ThemeProvider>
</SlotFillProvider>
```

## Contribution

Opened as a follow-up to #81112, the change was reviewed and merged with co-authorship from @aduth. It was subsequently cherry-picked to the `wp/7.1` branch for the upcoming release. The review thread was brief, confirming the token-seeding approach and `isRoot` behavior for portaled UI without proposing alternative implementations.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

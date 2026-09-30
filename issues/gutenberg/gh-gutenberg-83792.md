# #83792: Menu: Guard against nesting Group and RadioGroup

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] UI`
- **Merged:** [`831b7b1`](https://github.com/WordPress/gutenberg/commit/831b7b128b375b86315313234aec45dfef50dca6)
- **Discussion:** [#83792](https://github.com/WordPress/gutenberg/pull/83792) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`Menu.Group` and `Menu.RadioGroup` in `@wordpress/ui` now throw a development-only error when one is nested inside the other. `Menu.RadioGroup` already renders a group, so an extra `Menu.Group` is redundant and can attach a `Menu.GroupLabel` to the wrong group. The PR also removes the redundant `Menu.Group` wrappers from the editor's mode switcher and preview dropdown. Production rendering is unchanged.

## Impact

- **Plugin & theme developers using `@wordpress/ui` `Menu`:** In development builds, `<Menu.Group><Menu.RadioGroup/></Menu.Group>` and `<Menu.RadioGroup><Menu.Group/></Menu.RadioGroup>` now throw. This applies even when the nesting happens through wrapper components, so existing code that does this will break in dev. Fix it by removing the extra `Menu.Group` and putting `Menu.GroupLabel` inside `Menu.RadioGroup`.
- **Site owners / end users:** No visible change. The mode switcher (Visual/Code) and the preview device menu behave as before. The Editor label is now associated with its radio group.
- **Production builds:** No behavior change. The guards are skipped when `process.env.NODE_ENV === 'production'`, and the nested groups still render.
- **Sibling groups and separate menus:** Sibling `Group` and `RadioGroup` elements still work. The check resets at `Menu.Root` and `Menu.SubmenuRoot`, so groups in separate menus or submenus are not flagged.

## Technical details

The PR adds a `MenuGroupContext` in `packages/ui/src/menu/context.tsx`. It holds `'group' | 'radio-group' | null`, with a `useMenuGroupContext()` hook.

- `group.tsx`: reads the context. Outside production, it throws `Menu.Group: Cannot be nested inside Menu.RadioGroup...` if the parent is `'radio-group'`. It wraps `_Menu.Group` in `MenuGroupContext.Provider value="group"`.
- `radio-group.tsx`: does the mirror-image check. It throws `Menu.RadioGroup: Cannot be nested inside Menu.Group...` if the parent is `'group'`, and provides `'radio-group'`.
- `root.tsx` and `submenu-root.tsx`: wrap their children in `MenuGroupContext.Provider value={ null }` to reset the check.
- Because detection uses React context, it catches nesting through wrapper components.

Call-site change, for example in `packages/editor/src/components/mode-switcher/index.jsx`:

```jsx
// Before
<Menu.RadioGroup ...>
	<Menu.Group>
		<Menu.GroupLabel>{ __( 'Editor' ) }</Menu.GroupLabel>
		{ /* RadioItems */ }
	</Menu.Group>
</Menu.RadioGroup>

// After
<Menu.RadioGroup ...>
	<Menu.GroupLabel>{ __( 'Editor' ) }</Menu.GroupLabel>
	{ /* RadioItems */ }
</Menu.RadioGroup>
```

`preview-dropdown/index.jsx` gets the same `Menu.Group` removal. JSDoc on `Group`, `RadioGroup` and `GroupLabel`, plus `usage-guidelines.mdx`, now document placing `GroupLabel` inside `RadioGroup`. They also note that combining the two through the `render` prop counts as nesting. Jsdom tests cover both nesting directions through wrappers, sibling groups, separate roots and submenus, and unchanged production rendering via `vi.stubEnv('NODE_ENV', 'production')`. CHANGELOG entries were added for `packages/ui` and `packages/editor`.

## Contribution

This is a follow-up to #82968, opened by @ciampo after the nesting pattern "needed repeated corrections during Menu migrations". @mirka is credited in the props list. The PR description states it was implemented and verified with OpenAI Codex. The discussion shows only bot comments (props and bundle-size/perf reports), with no design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

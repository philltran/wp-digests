# #82346: Menu: Align selection indicators and prefix icons with item labels

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] DataViews`, `[Package] UI`
- **Merged:** [`b077718`](https://github.com/WordPress/gutenberg/commit/b077718b06f8e248575cb9dc4f77b3e8981de309)
- **Discussion:** [#82346](https://github.com/WordPress/gutenberg/pull/82346) · 13 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` `Menu` now top-aligns selection indicators (checkmarks, radio dots) and prefix icons with the first label line, rather than centering them against the full label plus description block. The PR adds a new public `Menu.PrefixIcon` component (24px default) that bakes in the vertical margin correction, and adopts it in DataViews table column menus and the editor's More menu.

## Impact

- **Plugin/theme developers using `@wordpress/ui` `Menu`:**
  - `Menu.PrefixIcon` is a new API for SVG icons in the `prefix` slot; using it is opt-in.
  - Existing menus get the new top-aligned layout and shared label line-height automatically. Single-line items keep their height, but leading visuals in items with descriptions or wrapped labels will visually shift up.
  - Arbitrary non-icon `prefix` content keeps its natural height. It aligns to the top of the label in described items and stays vertically centered in single-line items.
- **Editor/plugin authors using `PluginMoreMenuItem`:** Dashicon slugs and custom icon components still render via the `@wordpress/components` `Icon`. Only SVG elements (`svg` or `@wordpress/primitives` `SVG`) go through `Menu.PrefixIcon`.
- **Package maintainers:** `@wordpress/editor` gains a new dependency on `@wordpress/primitives` (`package.json`, both tsconfigs, lockfile).
- No breaking changes or deprecations.

## Technical details

**New component:** `packages/ui/src/menu/prefix-icon.tsx` exports a `forwardRef<SVGSVGElement, PrefixIconProps>` component, re-exported from `menu/index.ts` as `Menu.PrefixIcon`. It wraps `Icon` with `size = 24` by default, merges `styles['prefix-icon']` into `className`, and sets the custom property `--_wp-ui-menu-prefix-icon-size` (`${size}px`) via `style`. The CSS uses this together with a shared `--_wp-ui-menu-item-label-line-height` (set on `.list` from `--wpds-typography-line-height-sm`) to compute the margin that centers the icon on the first label line. The prefix slot already hides its contents from assistive technology.

**Selection indicators:** `checkbox-item.tsx` now passes `className={ styles['checkbox-selection-icon'] }` to the check `Icon`, applying an extra 1px downward optical offset. Indicators use a wrapper one label line high. Items share a top-aligned layout (changes in `style.module.css`; that part of the diff was truncated, so the exact rules are not shown here).

**More menu adapter** (`more-menu-item.tsx`):

```tsx
// before
const prefix = icon ? <WCIcon icon={ icon } /> : undefined;

// after
prefix =
	isValidElement( icon ) && ( icon.type === 'svg' || icon.type === SVG ) ? (
		<Menu.PrefixIcon icon={ icon } className={ icon.props.className } />
	) : (
		<WCIcon icon={ icon } />
	);
```

**DataViews:** `column-header-menu.tsx` swaps `Icon` for `Menu.PrefixIcon` on the filter, move left/right, and hide items.

**Other:** Storybook stories migrated to `Menu.PrefixIcon`, `usage-guidelines.mdx` gains an "Add prefix icons" section, a jsdom test file covers sizing, prop/ref forwarding and a11y, and CHANGELOGs were updated for ui, dataviews and editor. The `ui` CHANGELOG diff appears to add a duplicate `Breadcrumb` entry line.

## Contribution

Opened by @ciampo as a follow-up to PR #82321, first with top alignment only. Design feedback from @jasmussen, @fcoveram and @Mamaduka weighed three options for prefix icons: 20px icons to match the label line height, 24px icons with a negative vertical margin, or plain top alignment. They favored keeping the 24px icon size and optimizing for the first-party icon set and Dashicons, since third-party content can't be controlled. That led to the dedicated `Menu.PrefixIcon` with the size and margin correction built in, plus a slight adjustment to the checkmark. @jasmussen also floated a 20px footprint without scaling the SVG, which @ciampo said could be a prop on `Icon` and is deferred to separate experimentation. The PR notes Codex was used to implement and verify the change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

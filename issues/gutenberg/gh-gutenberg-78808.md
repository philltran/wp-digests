# #78808: Icons: Redraw 35 prominent icons with consistent stroke widths

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] Block library`, `[Package] Block editor`, `[Package] Icons`, `[Package] Edit Site`, `has dev note`, `[Package] UI`, `[Feature] Icons`
- **Merged:** [`3ed03bc`](https://github.com/WordPress/gutenberg/commit/3ed03bc90db90455e090ad9c3f7d94df4e6c9a19)
- **Discussion:** [#78808](https://github.com/WordPress/gutenberg/pull/78808) · 61 comments · 8 reactions
- **Usefulness:** 4/5

## Summary

Thirty-five prominent icons in `@wordpress/icons` (e.g. `paragraph`, `image`, `link`, `info`, `help`, `time`, `star-*`, `cover`, `tip`) are redrawn as stroke-based SVGs instead of fills. They use `currentColor`, an intrinsic `style="fill: none"`, and `vector-effect="non-scaling-stroke"`, so stroke weight stays constant at 16, 24, and 32px. To make this work, the `Icon` component now merges `style` props, the SVG sanitizer for registered icons allows stroke attributes, and several `fill`-based stylesheets switch to `color`. This reintroduces #78774, which was reverted in #78854, with compatibility fixes.

## Impact

**Plugin & theme developers**
- Recolor these icons with CSS `color`. A `fill` prop or an ordinary CSS `fill` rule no longer replaces the intrinsic `fill: none`; pass `style={ { fill: value } }` for a deliberate fill override.
- CSS that targets these icons with `fill` (including `svg`/`path` selectors) may stop having a visible effect and should be switched to `color`, or `stroke` where appropriate. The in-repo fixes were to `Tip`, block-directory ratings, block variation picker, and `AddNewTemplate`.
- `ItemGroup` no longer applies `path { fill: currentColor }`. Paths without an explicit fill still inherit `currentColor`; paths with an explicit fill now keep it.
- `SidebarNavigationItem` no longer forces `style={ { fill: 'currentcolor' } }` on its icon.
- Mixing new bundled icons with an older externalized `wp.components.Icon` can drop the icons' intrinsic styles when an unrelated `style` prop is passed. Keep the paired packages up to date.
- Some icons changed slightly: `time-to-read` now uses the same clock style as `pending`, and the `star-*` icons have fully sharp corners.

**Icon block / server-side rendering**
- The Icon block deliberately keeps proportional stroke scaling for backward compatibility.
- Registered icons (`WP_Icons_Registry`) may now use `stroke*`, `vector-effect`, and `style` attributes on `svg`/`path`/`polygon`.

**Site owners**
- No action required beyond expecting slightly different icon rendering.

## Technical details

**Icon component (`packages/components/src/icon/index.tsx`)**: the `isValidElement` branch now runs first. It pulls `style` out of `additionalProps` and shallow-merges it over `icon.props.style` (consumer wins per property). The merged style is applied to both the `svg`/`SVG` path and the `cloneElement` path. No `style` prop is added when neither side has one. `SizeProps` gains `style?: CSSProperties`.

```jsx
// icon has style={{ fill: 'none' }}
<Icon icon={ strokeIcon } style={ { marginInlineStart: 4 } } />
// before: intrinsic fill:none replaced by consumer style
// after:  fill:none + margin-inline-start:4px
```

**Sanitizer (PHP)**
- `lib/compat/wordpress-7.0/class-wp-icons-registry.php` adds `gutenberg_get_allowed_icon_svg_tags()`. It adds `style`, `stroke`, `stroke-width`, `stroke-linecap`, `stroke-linejoin`, `stroke-miterlimit`, and `vector-effect` to `svg`, `path`, and `polygon`, plus `fill`, `fill-rule`, and `clip-rule` on `svg`.
- The compat `WP_Icons_Registry::sanitize_icon_content()` now calls it.
- `lib/class-wp-icons-registry-gutenberg.php` overrides `sanitize_icon_content()` (untyped, to match core's parent signature) so the WP 7.0 parent-class path and the Gutenberg fallback both allow the attributes.
- A `backport-changelog/7.2/12197.md` entry is added.

**Icon block**
- `packages/block-library/src/icon/index.php`: `render_block_core_icon()` now merges the block's generated CSS and the rotation CSS into any existing `style` attribute on the `<svg>` (trimming trailing `;`) rather than overwriting it, so the intrinsic `fill: none` survives. Block styles come last and win on conflicts.
- `icon/style.scss`: `:where(.wp-block-icon) svg :where(path, rect, circle, ellipse, line, polygon, polyline) { vector-effect: none; }` restores proportional stroke scaling.

**Stylesheets**
- `block-directory` ratings, `Tip`, and the block variation picker (`content.scss`) switch from `fill` to `color`. The variation picker keeps a non-`!important` `fill` fallback for third-party icons that don't use `currentColor`, with `color` carrying the `!important`.
- `ItemGroup` (`style.module.scss`) drops `path` from the `svg, path { fill: currentColor }` rule.

**Other**
- Block-icon snapshot updated to the new `image` path with `stroke="currentColor"`, `stroke-width="1.5"`, `style="fill: none;"`, and `vector-effect="non-scaling-stroke"`.
- `package-lock.json` adds `@testing-library/jest-dom` and `@testing-library/react` dev dependencies for one package.
- CHANGELOG entries are added for block-directory, block-editor, block-library, components, and edit-site.
- Part of the diff was truncated in the source material, so the full icon SVG changes and `edit-site` code changes are not reviewed here.

## Contribution

This is a re-land: the first attempt (#78774) was reverted in #78854, and this PR carries compatibility fixes for icon styling and server-rendered SVGs. In review, @jameskoster suggested folding in low-hanging fixes from #65786 (endcap roundness, pixel-grid alignment, footprint consistency) and warned that existing `fill`-targeting CSS would behave differently with stroked icons. @jasmussen said he had already fixed pixel-grid issues and unified off-size circles, was already seeing the CSS breakage, and would defer further icon refinements to follow-ups once the stroke baseline landed. He also noted that hand-redrawing exposed minor pre-existing vector inaccuracies, mostly invisible at 24px. The PR carries a `has dev note` label, and the description states AI tools were used for the to-do list extraction and branch maintenance.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

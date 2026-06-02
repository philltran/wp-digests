# #77773: Make `@wordpress/nux` a no-op compatibility package

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `Needs Testing`, `[Type] Breaking Change`, `Needs Dev Note`, `[Package] Base styles`
- **Merged:** [`913ad1b`](https://github.com/WordPress/gutenberg/commit/913ad1b16bc6bf8e97a7b18efea5d7b99e355db1)
- **Discussion:** [#77773](https://github.com/WordPress/gutenberg/pull/77773) · 10 comments · 0 reactions

## Summary

`@wordpress/nux`, deprecated since WordPress 5.4, has been converted into a no-op compatibility shim rather than being removed outright. The `DotTip` component now renders nothing, all `core/nux` data store selectors return inert values, and all actions are still callable but have no runtime effect. The package entry point, script handle, and data store registration are preserved to avoid breaking existing imports, `wp_enqueue_script` calls, and older WordPress-Develop upgrade workflows. The change trims approximately 5 kB from the total Gutenberg build.

## Impact

**Plugin & theme developers**
- **Breaking change:** `<DotTip>` silently renders `null`. Any UI relying on it will disappear with no console error.
- **Breaking change:** `select('core/nux').isTipVisible()` and `select('core/nux').areTipsEnabled()` always return `false`; `getAssociatedGuide()` always returns `null`. Conditional logic gated on these selectors will never enter the truthy branch.
- All `dispatch('core/nux')` action calls (`triggerGuide`, `disableTips`, `enableTips`, etc.) remain valid but are no-ops.
- `wp.data.select('core/nux')` does not throw; the store is still registered.

**Hosting & platform / build pipelines**
- **Breaking change:** The `.nux-dot-tip` stacking-context entry has been removed from the `z-index()` SCSS helper in `@wordpress/base-styles`. Any custom SCSS that references `z-index( dot-tip )` will fail to compile.
- `build/styles/nux/style.min.css` is now empty/removed; `style.css` and `style-rtl.css` are reduced to ~118 bytes. Sites enqueuing the `wp-nux` style handle receive a near-empty stylesheet.

**No action required** for projects that do not import or enqueue `@wordpress/nux`.

## Technical details

**`DotTip` component** (`packages/nux/src/components/dot-tip/index.js`): updated to return `null`, rendering nothing regardless of props.

**`core/nux` selectors** (`packages/nux/src/store/selectors.js`):
- `isTipVisible()` → hardcoded `return false`
- `areTipsEnabled()` → hardcoded `return false`
- `getAssociatedGuide()` → hardcoded `return null`

**`core/nux` actions** (`packages/nux/src/store/actions.js`): remain exported and dispatchable but produce no state side-effects.

**`@wordpress/base-styles` z-index map**: the `dot-tip` entry removed from the `$z-layers` map (or equivalent SCSS index helper). This is reflected in the `[Package] Base styles` label on the PR.

**Build output delta:**
```
build/scripts/nux/index.min.js      909 B    −1.01 kB  (−52.73%)
build/styles/nux/style.css          118 B    −1.38 kB  (−92.13%)
build/styles/nux/style-rtl.css      118 B    −1.36 kB  (−92.03%)
build/styles/nux/style.min.css        0 B    removed
build/styles/nux/style-rtl.min.css    0 B    removed
```

The `core/nux` store is still registered in `wp.data`; calling `wp.data.select('core/nux')` does not throw. Existing consumers can import any public export from `@wordpress/nux` without a runtime error.

## Contribution

Opened and merged by @mirka (commit `913ad1b`). @tyxla noted the removal was overdue. @peterwilsoncc flagged that a prior removal attempt (tracked in issue #52444) broke the NPM-based upgrade script used by WordPress-Develop for security releases; he noted WordPress 7.0 will use a new integration script but that security-update branches for WordPress 6.9 and earlier would still require validation against those workflows. Co-authors credited in the merge include @ciampo, @peterwilsoncc, @tyxla, and @aaronjorbin.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

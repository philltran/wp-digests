# #81618: Admin UI: Fix the primary color of the Fresh admin color scheme

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `Design System`, `Backport to WP 7.1 Beta/RC`
- **Merged:** [`d3c65d3`](https://github.com/WordPress/gutenberg/commit/d3c65d3a7edc5728d173309e3ca093c84d301438)
- **Discussion:** [#81618](https://github.com/WordPress/gutenberg/pull/81618) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `fresh` entry in `@wordpress/admin-ui`'s admin theme color map was seeded with `#3858e9` — the Modern scheme's primary — so users on the default Fresh admin color scheme saw the Modern accent throughout the block editors instead of the classic WordPress blue. This PR changes that single map entry to `#007cba`, matching what Fresh's stylesheets have always produced. The regression surfaced because Fresh has no `body.admin-color-fresh` rule in the `wordpress-admin-schemes()` Sass mixin, so its accent resolves to the `:root` fallback in `_default-custom-properties.scss` (`#007cba`), while `ThemeProvider` emits `--wp-admin-theme-color` as an inline style that overrode that fallback with the wrong seeded value.

## Impact

- **Site owners / editors:** Fresh (the default admin color scheme) once again renders the classic `#007cba` blue for accents — Save button, inserter, focus rings — in the post editor, site editor, widgets screen, and the Customizer widgets panel. No action required.
- **Plugin & theme developers:** If you read `--wp-admin-theme-color` or call `getAdminThemeColors()` from `@wordpress/admin-ui`, the value returned for `fresh` changes from `#3858e9` to `#007cba`. Anything that hardcoded or snapshot-tested the old value under Fresh needs updating; visual regression baselines captured against Gutenberg 7.0/7.1 pre-release under Fresh will shift.
- **No other scheme is affected** — `modern`, `midnight`, `coffee`, `ocean`, etc. already matched their `wordpress-admin-schemes()` values.
- **Known remaining gap on `trunk`:** `Popover` and `Modal` content in the site editor still shows the Modern accent for *every* scheme, because the outer `isRoot` `ThemeProvider` that portaled UI needs (#81296) landed only on the 7.1 release branch; portaled content there falls back to the hardcoded `#3858e9` in `@wordpress/theme`'s prebuilt `design-tokens.css`. That is tracked separately (#81183).

## Technical details

The functional change is one line in `packages/admin-ui/src/admin-theme-colors/index.ts`, in the `ADMIN_THEME_COLORS` map consumed by `getAdminThemeColors()`:

```diff
 const ADMIN_THEME_COLORS = new Map< string, AdminThemeColors >( [
 	[ 'modern', DEFAULT_THEME_COLORS ],
-	[ 'fresh', { primary: '#3858e9', background: '#25292b' } ],
+	[ 'fresh', { primary: '#007cba', background: '#25292b' } ],
 	[ 'midnight', { primary: '#cf4339', background: '#3d4042' } ],
```

The `background` value is untouched; only `primary` moves. The mechanism of the bug: `wordpress-admin-schemes()` emits `body.admin-color-*` rules for each scheme but has no rule for Fresh, so Fresh's accent has always come from the `:root` fallback in `_default-custom-properties.scss` (`#007cba`). #78397 introduced the JS color map and wired it into the site editor; #81112, #81173, and #81174 did the same for the post editor, widgets editor, and Customizer widgets. Because `ThemeProvider` writes `--wp-admin-theme-color` as an *inline* style, the seeded (wrong) map value won over the `:root` fallback in exactly those four surfaces. A `packages/admin-ui/CHANGELOG.md` entry under **Bug Fixes** accompanies the change. Compressed-size impact is +13 B total across `edit-post`, `edit-site`, `edit-widgets`, `customize-widgets`, and the boot module.

## Contribution

The mis-mapping traced back to a reasonable-looking wrong inference: @fushar, who added the map entry, confirmed in review that they had set Fresh's primary to Modern's after observing that the newer React-based admin screens (e.g. **Appearance → Fonts**) rendered the same accent for both schemes in 7.0, and assumed that was intentional — not realizing the site editor already rendered Fresh correctly. @mirka reviewed and noted that the portaled-UI (`Popover`/`Modal`) side of the problem means the "inverted" `ThemeProvider` approach from the 7.1 branch has to come back to `trunk`, opening #81653 to do so. The automated cherry-pick to `wp/7.1` hit a conflict, so the backport was left to be raised manually. The author disclosed that the investigation, the fix, and the changelog entry were drafted with Claude Code and reviewed by hand.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #79451: Icons: ship required icons for admin bar and menu with public: false

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`, `[Package] Icons`
- **Merged:** [`cbbd2b3`](https://github.com/WordPress/gutenberg/commit/cbbd2b39f40b7ffd85cb89dc16bfe687249b9ab7)
- **Discussion:** [#79451](https://github.com/WordPress/gutenberg/pull/79451) · 13 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Ten icons (`brush`, `dashboard`, `link`, `media`, `page`, `pin`, `plugins`, `sites`, `tool`, `update`) are marked `public: false` in the `@wordpress/icons` manifest. They now ship to WordPress Core in the `core` icon collection and can be resolved server-side with `wp_get_icon()`, but they are hidden from the icons REST API and the Icon block's picker. They are intended to replace the old dashicons in the admin bar and admin menu.

## Impact

- **Core / admin UI contributors:** These ten icons, plus `core/wordpress` (from #82634), are now available to `wp_get_icon()` once the Gutenberg manifest is synced into Core. This is the prerequisite for the related Core PR (wordpress-develop #12270), which replaces dashicons in the admin bar and menu.
- **Plugin & theme developers:** Once synced into Core, `wp_get_icon( 'core/brush' )` and the other slugs should resolve server-side. They are not exposed via the icons REST API, so JS or headless consumers cannot list them there. Nothing is required from you; the capability is usable only after Core picks up the change.
- **Site owners / editors:** No visible change in the Icon block. The new icons do not appear in its search, so post content is not cluttered with admin-oriented icons like `dashboard` or `sites`.
- **Headless & REST consumers:** No change to the icons REST API output.
- No breaking changes or deprecations.

## Technical details

The diff touches three files in `packages/icons`:

- `src/manifest.json`: adds `"public": false` to the ten entries. Per the CHANGELOG (from #82634), `public` is tri-state. Omitted means the icon stays in the JS library only. `true` ships it to Core, exposes it via the icons REST API, and makes it selectable in the Icon block. `false` ships it and registers it in the `core` collection for `wp_get_icon()`, hiding it from the REST API and the Icon block.
- `src/manifest.php`: adds ten new entries, inserted alphabetically. Each has a translatable label via `_x( '…', 'icon label', 'gutenberg' )`, a `filePath` of `library/<slug>.svg`, and `'public' => false`. This is the PHP manifest that Core consumes.
- `CHANGELOG.md`: extends the existing entry about `wordpress` shipping as non-public to list all eleven icons, crediting #82634 and #79451.

```json
{
  "slug": "dashboard",
  "label": "Dashboard",
  "filePath": "library/dashboard.svg",
  "public": false
}
```

No new hooks, functions, or schema changes are introduced. The PR only uses the `public: false` mechanism introduced in #82634.

## Contribution

The PR initially met resistance from @t-hamano, who objected that publishing icons makes them selectable in the Icon block, which is undesirable for admin-menu icons like `dashboard`. He proposed a way to register icons without exposing them in the block. @tyxla responded that a `public` flag had already been agreed on, and the discussion converged on a tri-state `public` (omitted / `true` / `false`), with @mcsf's input requested. That mechanism landed separately in #82634, and this PR was reworked to use it. The description notes the changes were generated with Claude Opus and double-checked by the author.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

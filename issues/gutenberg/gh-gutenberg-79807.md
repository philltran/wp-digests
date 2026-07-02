# #79807: Omnibar: move the 'site icon in admin bar' feature from experiment to 7.1 compat

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`, `Backwards Compatibility`
- **Merged:** [`1040e42`](https://github.com/WordPress/gutenberg/commit/1040e4267db40194250f40f1bcea607ec4d17ddc)
- **Discussion:** [#79807](https://github.com/WordPress/gutenberg/pull/79807) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request moves the 'site icon in admin bar' feature from the experimental `gutenberg-omnibar` flag into the WordPress 7.1 compatibility layer. The change ensures that a configured site icon replaces the default home/odometer dashicon in the admin bar, maintaining visual consistency across the Post and Site Editors even when this Gutenberg release is installed on pre-7.1 WordPress versions.

## Impact

- **Plugin & Theme Developers**: No immediate code changes required. If you were gating logic behind `gutenberg_is_experiment_enabled( 'gutenberg-omnibar' )`, that flag is now obsolete as the behavior ships unconditionally (subject to standard filters).
- **Hosting & Platform Teams**: Ensures consistent admin bar rendering across WordPress 6.x+ and 7.1 when paired with this Gutenberg release. No migration or configuration changes needed.
- **Site Owners/Admins**: Sees the configured site icon in the admin bar instead of the generic dashboard icon, provided `is_network_admin()` or `is_user_admin()` is false and `wp_admin_bar_show_site_icons` is not filtered to false.

## Technical details

- Removed `lib/experimental/omnibar/load.php` and its `'gutenberg-omnibar'` experiment definition from `lib/experimental/experiments/load.php`.
- Added `lib/compat/wordpress-7.1/admin-bar.php`, introducing:
  - `gutenberg_admin_bar_site_icon()` hooked to `admin_bar_menu` (priority 31), which early-returns for network/user admin contexts, gates on the `wp_admin_bar_show_site_icons` filter and `has_site_icon()`, then replaces the `'site-name'` node title with a 32px `<img>` (with optional `srcset` for 64px) and applies a `.has-site-icon` class.
  - `gutenberg_admin_bar_site_icon_styles()` hooked to `admin_bar_init`, injecting inline CSS for `.site-icon` sizing, gap spacing, and mobile adjustments.
- Updated `lib/load.php` to `require` the new compat file instead of the deleted experimental loader.
- The logic shifted from checking `gutenberg_is_experiment_enabled()` to baking the behavior into the 7.1 compatibility layer, making it default while preserving backward compatibility via standard WordPress filters.

## Contribution

Opened and merged by @fushar (with co-authorship from @t-hamano per the commit metadata). The PR was a straightforward port of a core WordPress commit (#11781) to the 7.1 compat layer, with reviewer feedback confirming merge readiness. Discussion remained brief, focusing primarily on acknowledging unrelated test flakiness reports rather than debating implementation details.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

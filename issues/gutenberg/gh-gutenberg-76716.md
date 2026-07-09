# #76716: Add backport for WP_ALLOW_COLLABORATION

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ingeniumed
- **Labels:** `[Type] Enhancement`, `Needs Dev Note`, `[Feature] Real-time Collaboration`, `Backported to WP Core`
- **Merged:** [`4f4a7b1`](https://github.com/WordPress/gutenberg/commit/4f4a7b183063af63646edfb7f5e6168d74395804)
- **Discussion:** [#76716](https://github.com/WordPress/gutenberg/pull/76716) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Backports WordPress core's Real-Time Collaboration (RTC) configuration to the Gutenberg 7.0 branch, introducing the `WP_ALLOW_COLLABORATION` constant and new helper functions. This shifts the default state of the `wp_collaboration_enabled` option from `false` to `true`, effectively enabling real-time collaboration out of the box while preserving an environment-level kill switch.

## Impact

- **Hosting & Platform Teams**: Can permanently disable RTC at the server level by defining `WP_ALLOW_COLLABORATION` as `false` or setting an environment variable with the same value and content.
- **Plugin & Block Developers**: Replace direct calls to `get_option( 'wp_collaboration_enabled' )` with `wp_is_collaboration_enabled()` to respect the constant's restriction during post locking, autosaving, and UI injection.
- **Site Owners/Admins**: Will see RTC enabled by default; the writing settings page now displays a warning notice instead of the toggle when the constant explicitly forbids it.
- No immediate action required for sites not using Real-Time Collaboration or that hardcode the option value in their environments.

## Technical details

The diff modifies `lib/compat/wordpress-7.0/collaboration.php` and `lib/compat/wordpress-7.0/class-gutenberg-rest-autosaves-controller.php`. It introduces two new functions:

- `wp_is_collaboration_allowed()`: Checks the `WP_ALLOW_COLLABORATION` constant, falling back to `getenv( 'WP_ALLOW_COLLABORATION' )`. Explicitly handles string values (e.g., `'false'`) before returning a boolean, defaulting to `true` if neither is defined.
- `wp_is_collaboration_enabled()`: Returns `( wp_is_collaboration_allowed() && (bool) get_option( 'wp_collaboration_enabled' ) )`.

The implementation replaces direct option reads across the codebase. **Before**:
```php
$is_collaboration_enabled = get_option( 'wp_collaboration_enabled' );
```
**After**:
```php
$is_collaboration_enabled = wp_is_collaboration_enabled();
```

The admin UI in `gutenberg_register_real_time_collaboration_setting()` now wraps the checkbox in a conditional that checks `wp_is_collaboration_allowed()`, rendering a warning notice when collaboration is explicitly disallowed. A backport changelog file was also added under `backport-changelog/7.0/11311.md`.

## Contribution

Opened by @ingeniumed to align the Gutenberg 7.0 branch with core's #11311 implementation. Co-authored by @chriszarate and originally authored by @alecgeatches, who designed the constant and environment variable handling in WordPress core. The PR was merged as commit `4f4a7b1` after finalizing the default state shift and ensuring the environment variable fallback strictly handles string inputs.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

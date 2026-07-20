# Users: Improve interfaces for deleting users.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Joe Dolson
- **Committed:** 2026-07-10
- **Commit:** [`3ec2726426`](https://github.com/WordPress/wordpress-develop/commit/3ec27264267f3aeae8c3485cb9012441c2a88d12)
- **Usefulness:** 4/5

## Summary

The user deletion interface has been rewritten to enforce per-site content decisions in multisite, improve form accessibility, and add strict client- and server-side validation. The form now excludes users marked for deletion from reassignment dropdowns, syncs radio and select controls, and replaces disabled-submit patterns with a focusable, screen-reader-friendly error summary. Server-side validation was added to prevent invalid reassignments when JavaScript is bypassed.

## Impact

- **Multisite administrators:** Must explicitly assign or delete content per user per site before the form submits.
- **Plugin & theme developers:** Code hooking into `delete_user_form` receives the updated `$users` array. The previously inline `delete_users_add_js()` function is now deprecated in favor of the shared `common.js` module.
- **Hosting & platform teams:** No configuration changes required, but custom scripts that bypass the admin UI or submit `users.php?action=dodelete` will be rejected with a `missing_reassign` redirect if reassignment IDs are invalid.
- **Action required:** Remove custom code calling `delete_users_add_js()`. Audit any custom `delete_user_form` integrations for the new field structure and `required` attributes.

## Technical details

The rewrite lands in `src/wp-admin/includes/ms.php` (`confirm_delete_users()`), `src/js/_enqueues/admin/common.js`, and `src/wp-admin/network/users.php`. The form now uses the `.delete-and-reassign-users-form` class and loops through each user/site combination, querying `$wpdb` directly for `post_author` and `link_owner` records to conditionally render a “no content” notice or a full `<fieldset>` with radio/`<select>` controls. The `wp_dropdown_users()` call now passes `'exclude' => $users` to prevent self-reassignment, and the new `users_have_additional_content` filter allows early termination of DB checks. A shared JS module (tagged `@since 7.1.0`) attaches to the form, listens for `submit` and `change` events, and renders an accessible error summary using `wp.a11y.speak()`. `src/wp-admin/includes/user.php` removed the legacy inline script, and `deprecated.php` marked `delete_users_add_js()` as deprecated. `network/users.php` adds defense-in-depth validation, redirecting to `network/admin/users.php?action=delete&error=missing_reassign` if `$_POST['blog']` contains a reassign value `< 1`.

## Contribution

Authored by Joe Dolson and committed on 2026-07-10 (r62688). The changes consolidated work from PRs #10502 and #12467 to resolve accessibility and UX gaps tracked in #56914. Key contributors included stefan.velthuys, ocean90, hellofromTonya, afercia, and krokodok. Early iterations explored disabling the submit button, but the team shifted to a focusable error-summary pattern in `common.js` after testing revealed assistive technology compatibility issues and maintenance overhead across the single-site and multisite form variations.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

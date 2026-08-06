# REST API: Pass the missing `$is_update` param to the `wp_creating_autosave` action.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Weston Ruter
- **Committed:** 2026-07-29
- **Commit:** [`2583553006`](https://github.com/WordPress/wordpress-develop/commit/258355300666058b0851f4bfd50b9a596c2cd22c)
- **Usefulness:** 3/5

## Summary

The `wp_creating_autosave` action now correctly receives its second `$is_update` parameter when triggered via the REST API. Since WordPress 6.4.0, this parameter has been documented but was omitted in `WP_REST_Autosaves_Controller::create_post_autosave()`, which could trigger fatal errors in callbacks expecting the argument. This commit aligns the REST API autosave creation path with the legacy `wp_create_post_autosave()` function by passing `true` to indicate an update operation.

## Impact

- **Plugin & theme developers:** Callbacks hooked to `wp_creating_autosave` that rely on the second parameter will no longer receive `null` or trigger a fatal error when autosaves are created via the REST API.
- **No action required** for most users; existing code that ignores the second parameter continues to work unchanged.
- If you explicitly type-hint or count arguments for this action, ensure your callbacks accept the second `$is_update` boolean.

## Technical details

The change modifies a single `do_action()` call in `src/wp-includes/rest-api/endpoints/class-wp-rest-autosaves-controller.php` within `WP_REST_Autosaves_Controller::create_post_autosave()`. Previously, the action fired with only the `$new_autosave` array. The diff adds `true` as the second argument to match the signature used by `wp_create_post_autosave()`:

```php
// Before
do_action( 'wp_creating_autosave', $new_autosave );
// After
do_action( 'wp_creating_autosave', $new_autosave, true );
```

This ensures the `$is_update` parameter is consistently passed across both the legacy admin path and the REST API path for autosave creation.

## Contribution

Weston Ruter submitted this as a targeted follow-up to the original 6.4.0 implementation after PHPStan static analysis surfaced the missing argument during development of related extensions. The change was prepared as a subset of PR #12022 and merged without debate, directly mirroring the legacy `wp_create_post_autosave()` signature to resolve the omission.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

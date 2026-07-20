# REST API: sub-requests must always use dispatch.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jonathan Desrosiers
- **Committed:** 2026-07-17
- **Commit:** [`fa72c12879`](https://github.com/WordPress/wordpress-develop/commit/fa72c12879fbfda17d450fc1fd919f698f549f4d)
- **Usefulness:** 4/5

## Summary

WordPress now strictly prevents nested REST request cycles by short-circuiting when a dispatch is already in progress. Guards were added to `rest_api_loaded()` and `WP_REST_Server::serve_request()` to detect `$GLOBALS['wp_rest_server']->is_dispatching()` and abort any new top-level cycle. This enforces that internal sub-requests must use the existing server’s `dispatch()` method, eliminating double execution of routes, hooks, and potential fatal errors from nested `die()` calls.

## Impact

- **Plugin & theme developers**: Internal REST calls that trigger a fresh server bootstrap during an active dispatch will now short-circuit. Sub-requests must use `$server->dispatch()` (or `$GLOBALS['wp_rest_server']->dispatch()`) to avoid silent failures or skipped response handling.
- **Headless & REST consumers**: No direct impact unless you rely on nested server bootstrapping or custom request routing.
- **Hosting & platform teams**: No configuration or migration required; the re-entry guard is automatic and backward-compatible for standard dispatch usage.

## Technical details

The commit modifies two core files to add re-entry guards:

1. In `src/wp-includes/rest-api.php`, `rest_api_loaded()` now checks early in execution:
   ```php
   if ( isset( $GLOBALS['wp_rest_server'] ) && $GLOBALS['wp_rest_server']->is_dispatching() ) {
       return;
   }
   ```
   This prevents re-entering the REST bootstrap loop and avoids a trailing `die()` call.

2. In `src/wp-includes/rest-api/class-wp-rest-server.php`, `WP_REST_Server::serve_request()` gains an immediate check:
   ```php
   if ( $this->is_dispatching() ) {
       return false;
   }
   ```
   Both guards explicitly refuse to start a fresh top-level REST cycle while another is active, formalizing that internal sub-requests must route through `WP_REST_Server::dispatch()` on the already-initialized server instance.

## Contribution

Committed by Jonathan Desrosiers on 2026-07-17 as a Trac sync. Props credited to xknown, johnbillion, joehoyle, jorbin, sergeybiryukov, ehtis, and desrosj. The change formalizes an existing best practice by adding explicit re-entry guards to the REST bootstrap and server methods, preventing nested cycles without altering the public dispatch flow.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

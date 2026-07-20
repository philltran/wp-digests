# REST API: Ensure errors in batch requests propogate.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jonathan Desrosiers
- **Committed:** 2026-07-17
- **Commit:** [`1000ed1649`](https://github.com/WordPress/wordpress-develop/commit/1000ed164983a2733e974861aa7332234bf23d16)
- **Usefulness:** 3/5

## Summary

This commit fixes a bug in the REST API batch endpoint where validation errors for individual sub-requests were silently dropped from the response. When a batch request contains invalid items, those errors are now correctly collected and returned alongside successful results, ensuring clients receive complete error feedback.

## Impact

- **Headless & REST consumers:** Validation failures in `/wp-json/wp/v2/batch` requests will now be included in the response payload instead of being silently discarded.
- **Plugin & theme developers:** No code changes required; existing integrations will simply receive more accurate error data for malformed batch requests.
- **Hosting & platform teams:** No action required.

## Technical details

The unified diff adds a single line to `serve_batch_request_v1()` in `wp-includes/rest-api/class-wp-rest-server.php`. Inside the loop that iterates over sub-requests, when `is_wp_error( $single_request )` evaluates true, the code now appends the error object to the `$matches` array (`$matches[] = $single_request;`). Previously, `$matches` only collected successfully processed requests, causing validation errors to be excluded from the final response array assembled for the client.

## Contribution

Committed by Jonathan Desrosiers on 2026-07-17. Props credited to xknown, sergeybiryukov, joehoyle, jorbin, clorith, johnbillion, and desrosj. The fix addresses a gap in the batch request handling loop where validation failures were tracked via `$has_error` and `$validation` but omitted from the final response assembly.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

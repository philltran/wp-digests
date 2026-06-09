# REST API: Fix `rest_is_integer()` returning false for large integers.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Weston Ruter
- **Committed:** 2026-06-08
- **Commit:** [`dbf0e21fea`](https://github.com/WordPress/wordpress-develop/commit/dbf0e21feacbe1fc8d14b126b06d3e58a4ab4117)

## Summary

The `rest_is_integer()` function in the REST API was rewritten to correctly identify integer-like values of any magnitude, resolving a PHP 8.4 regression where large integers were incorrectly flagged as non-integers. The previous implementation relied on `round( (float) $value ) === (float) $value`, which became unreliable due to a `round()` behavior change in PHP 8.4 that caused valid large integers to fail schema validation. This fix ensures REST endpoints no longer reject canonical integer strings or high-value numeric parameters that exceed previous precision thresholds.

## Impact

- **Plugin & Theme Developers**: Schemas of type `'integer'` will now correctly validate and sanitize large numeric strings (e.g., IDs exceeding `2^53` or Bigint fields) on PHP 8.4+. No code changes are required for existing implementations, but developers should note the function explicitly returns `true` for magnitude-agnostic canonical integer strings.
- **Hosting & Platform Teams**: Servers running PHP 8.4+ will see corrected REST validation behavior without requiring configuration updates or database migrations.
- **Headless & REST API Consumers**: Requests or responses containing large integer values will no longer be incorrectly rejected by the REST validator, preventing unexpected `400` validation errors for out-of-range-but-valid numbers.
- **Action Required**: None. This is a backward-compatible bug fix that restores expected validation behavior.

## Technical details

- **Modified Symbol**: `rest_is_integer( $maybe_integer ): bool` in `src/wp-includes/rest-api.php`
- **Behavior Shift**: The function was restructured from a single-line float comparison to a three-path check:
  1. Native integers are immediately accepted via `is_int()`.
  2. Canonical integer strings matching `/^\s*[+-]?[0-9]+\s*$/` short-circuit as valid, entirely bypassing float conversion.
  3. Remaining numeric types fall back to casting to `$float_value` and returning `floor( $float_value ) === $float_value`. This comparison is exact because a float equals its floor if and only if it possesses no fractional part, eliminating the previous `round()` regression while preserving historical decimal/scientific-notation behavior.
- **Before/After Snippet**:
  ```diff
- function rest_is_integer( $maybe_integer ) {
-     return is_numeric( $maybe_integer ) && round( (float) $maybe_integer ) === (float) $maybe_integer;
+ function rest_is_integer( $maybe_integer ): bool {
+     if ( is_int( $maybe_integer ) ) { return true; }
+     if ( is_string( $maybe_integer ) && preg_match( '/^\s*[+-]?[0-9]+\s*$/', $maybe_integer ) ) { return true; }
+     if ( ! is_numeric( $maybe_integer ) ) { return false; }
+     $float_value = (float) $maybe_integer;
+     return floor( $float_value ) === $float_value;
  }
  ```
- **Documentation & Tests**: The function docblock was updated to explicitly document `PHP_INT_MAX` limits and `(int)` cast saturation behavior. Test coverage in `tests/phpunit/tests/rest-api.php` expanded the `data_rest_is_integer()` provider to include scientific notation (`15e0`, `1.5e3`), large negatives, out-of-range floats (`PHP_INT_MAX + 1`), and explicitly asserts sanitization round-trips where behavior is defined.

## Contribution

The change was developed as PR #11893 and merged on 2026-06-08 by Weston Ruter, addressing regression introduced in PHP 8.4 reported in Ticket #65271. The diff reflects a deliberate shift from a fragile float-based `round()` check to a magnitude-agnostic early return for canonical strings, followed by an exact floor comparison for remaining numeric types. Test data was expanded to cover edge cases like scientific notation and out-of-range floats, with `null` sanitization assertions explicitly left for PHP's undefined conversion ranges.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

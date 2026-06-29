# Performance: avoid over-allocation in wp_is_numeric_array()

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Dennis Snell
- **Committed:** 2026-06-18
- **Commit:** [`9569f22875`](https://github.com/WordPress/wordpress-develop/commit/9569f228755792542f0e10047b991c82f5b07e5f)
- **Usefulness:** 3/5

## Summary

`wp_is_numeric_array()` in `src/wp-includes/functions.php` was refactored to eliminate eager memory allocation and replace it with an early-exit iteration pattern. Previous performance tracing identified that eagerly calling `array_keys()` and `array_filter()` caused significant overhead when validating arrays during page rendering. The updated implementation now short-circuits on the first string key, reducing runtime allocations while preserving identical boolean return behavior.

## Impact

- Plugin & theme developers: No action required. The public API, function signature, and return values remain unchanged; consumers only gain reduced memory overhead.
- Core internals/Third-party packages: Zero breaking changes or deprecations. Existing callers of `wp_is_numeric_array()` continue to function identically without migration steps.

## Technical details

The diff replaces the two-pass array mapping in `src/wp-includes/functions.php` with a single-pass iteration. Replaced code:
```php
$keys         = array_keys( $data );
$string_keys = array_filter( $keys, 'is_string' );
return count( $string_keys ) === 0;
```
New code uses:
```php
foreach ( $data as $key => $value ) {
	if ( is_string( $key ) ) {
		return false;
	}
}
return true;
```
The function signature now declares an explicit return type (`: bool`) and includes an updated docblock clarifying its distinction from PHP's native `array_is_list()`, alongside a `@phpstan-assert-if-true array<int, mixed> $data` annotation. Test updates in `tests/phpunit/tests/functions/wpIsNumericArray.php` validate early-exit behavior on sparse arrays and filtered lists with missing numeric keys.

## Contribution

Merged by Dennis Snell on 2026-06-18 following review in PR #12100 and Trac ticket #65467, developed during WCEU 2026 Contributor Day. Reviewers westonruter and yusufmudagal provided validation on the allocation trace findings. The change shipped as a direct algorithmic optimization after performance profiling pinpointed `wp_is_numeric_array()` as a notable allocation hotspot in render pipelines.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

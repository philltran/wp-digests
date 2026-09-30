# REST API: Split comma-separated methods in array form.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Weston Ruter
- **Committed:** 2026-09-18
- **Commit:** [`cef8ae398d`](https://github.com/WordPress/wordpress-develop/commit/cef8ae398dec880f84571c5b660c443b88d75d92)
- **Usefulness:** 3/5

## Summary

`WP_REST_Server::get_routes()` now splits comma-separated values inside an array-form `methods` argument, matching how the string form was already handled. Previously `array( WP_REST_Server::READABLE, WP_REST_Server::EDITABLE )` registered a literal `'POST, PUT, PATCH'` key, so POST, PUT and PATCH requests returned 404 with no notice. The commit also corrects the `get_routes()` docblock, which wrongly described a `$callback, $bitmask` shape, and adds `@phpstan-type` aliases for route handlers and endpoint args.

## Impact

**Plugin & theme developers**
- Routes registered with an array of `methods` that includes multi-method constants (`WP_REST_Server::EDITABLE`, `ALLMETHODS`) or comma-joined strings now match their HTTP methods. Handlers that were silently unreachable via those methods (404) will start receiving requests.
- Check any code that worked around the bug, for example by splitting the constants manually. It should keep working, since the result is the same normalized method map.
- Because previously dead methods become live, confirm that `permission_callback` and argument validation on such routes are correct before upgrading.

**Static analysis users**
- `get_routes()` now has a real return type (`array<non-empty-string, array<int, Route_Handler>>`), so PHPStan users no longer get `mixed` from the first offset.

**Site owners, hosting, and REST consumers**
- No action required. Only routes affected by the bug change behavior.

## Technical details

In `src/wp-includes/rest-api/class-wp-rest-server.php`, the normalization loop in `get_routes()` used to do `$methods = $handler['methods'];` for array input. It now iterates `array_filter( $handler['methods'], 'is_string' )` and merges `explode( ',', $method )` for each element. Non-string elements are dropped. The existing code then builds the `methods` map of method name to `true`.

```php
// Before: array elements were used as-is
'methods' => array( WP_REST_Server::READABLE, WP_REST_Server::EDITABLE )
// => keys: 'GET', 'POST, PUT, PATCH'  (the second never matches)

// After
// => keys: 'GET', 'POST', 'PUT', 'PATCH'
```

The rest of the diff is documentation only:
- The `get_routes()` `@return` description is rewritten. Handlers are lists of endpoint arguments, and route options are moved out to `get_route_options()`.
- `@phpstan-type Endpoint_Arg` and `Route_Handler` aliases are added on the class.
- `@phpstan-var` annotations are added on `$namespaces`, `$endpoints` and `$route_options`.
- The `rest_endpoints` filter docblock is updated to say the endpoints are as registered, before normalization.
- A `@phpstan-var` annotation is added after the loop to cover the by-reference mutation.

New tests in `tests/phpunit/tests/rest-api.php`:
- comma-separated values inside an array
- a multi-method constant inside an array
- dispatch of GET, POST, PUT and PATCH returning 200
- an empty array yielding an empty methods map

## Contribution

Developed in a PR by westonruter with props to moonmeister, and committed as a follow-up to r34928, which introduced the comma-split for the string form. It is tracked under #65905 and references related tickets #43744, #65616 and #65817.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

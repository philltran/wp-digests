# REST API: Allow-list ability schema response keywords

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Greg Ziółkowski
- **Committed:** 2026-06-02
- **Commit:** [`da7588be08`](https://github.com/WordPress/wordpress-develop/commit/da7588be089fbaeb05ecb4d79dca3cf11815698d)

## Summary

The `prepare_schema_for_response()` method in `WP_REST_Abilities_V1_List_Controller` has been switched from a deny-list approach to an allow-list approach when serializing ability `input_schema` and `output_schema` for REST clients. Previously, only three known WordPress-internal keys (`sanitize_callback`, `validate_callback`, `arg_options`) were stripped; any other non-standard keyword silently leaked through. The new implementation keeps only keys returned by `rest_get_allowed_schema_keywords()` plus a curated set of additional JSON Schema draft-04 keywords, stripping everything else — including WordPress-internal keywords like `context`, `readonly`, `example`, and `examples` that the old deny-list missed. Seven additional draft-04 keywords (`required`, `allOf`, `not`, `$ref`, `definitions`, `dependencies`, `additionalItems`) are explicitly preserved so clients can validate richer schemas.

## Impact

**Plugin & theme developers registering abilities:**
- **Action required if your ability schemas rely on previously-leaked keywords.** WordPress-internal keys that were silently passed through — `context`, `readonly`, `example`, `examples` — are now stripped from REST responses. Remove them from your schemas or accept that REST clients will no longer see them.
- `sanitize_callback`, `validate_callback`, and `arg_options` continue to be stripped (same as before).
- The additional draft-04 keywords `required`, `allOf`, `not`, `$ref`, `definitions`, `dependencies`, and `additionalItems` are **now preserved** in responses — abilities that use these keywords will have them correctly exposed for the first time.

**Headless & REST consumers:**
- Ability schema responses are now strictly JSON Schema draft-04-compatible. Any tooling that parsed or relied on the presence of `context`, `readonly`, `example`, or similar WordPress-internal fields will need to be updated.
- `$ref` and `definitions` are now reliably surfaced, enabling schema composition patterns on the client side.

**Hosting & platform / site owners:** No action required.

## Technical details

**File:** `src/wp-includes/rest-api/endpoints/class-wp-rest-abilities-v1-list-controller.php`

The private constant `INTERNAL_SCHEMA_KEYWORDS` (introduced in 7.0.0) has been replaced by `ADDITIONAL_ALLOWED_SCHEMA_KEYWORDS` (7.1.0). The old constant was a `array<string, true>` map used with `array_diff_key()` to subtract known bad keys. The new constant is a plain `string[]` of additional draft-04 keywords to add on top of `rest_get_allowed_schema_keywords()`:

```php
// Before (deny-list)
private const INTERNAL_SCHEMA_KEYWORDS = array(
    'sanitize_callback' => true,
    'validate_callback' => true,
    'arg_options'       => true,
);
$schema = array_diff_key( $schema, self::INTERNAL_SCHEMA_KEYWORDS );

// After (allow-list)
private const ADDITIONAL_ALLOWED_SCHEMA_KEYWORDS = array(
    'required', 'allOf', 'not', '$ref',
    'definitions', 'dependencies', 'additionalItems',
);
static $allowed_keywords = null;
$allowed_keywords ??= array_fill_keys(
    array_merge(
        rest_get_allowed_schema_keywords(),
        self::ADDITIONAL_ALLOWED_SCHEMA_KEYWORDS
    ),
    true
);
$schema = array_intersect_key( $schema, $allowed_keywords );
```

The allowed-keyword set is built once per request using a `static` variable with null-coalescing assignment (`??=`), avoiding repeated computation across recursive calls that walk nested sub-schemas (`properties`, `items`, `anyOf`, `oneOf`, `allOf`, etc.).

The REST route affected is `GET /wp-abilities/v1/abilities/{namespace}/{ability}` (and the collection endpoint). Both `input_schema` and `output_schema` fields in the response go through `prepare_schema_for_response()` recursively.

**Test file:** `tests/phpunit/tests/rest-api/wpRestAbilitiesV1ListController.php` — the existing test `test_internal_schema_keywords_stripped_from_response` was renamed to `test_unsupported_schema_keywords_stripped_from_response` and extended to assert that `example`, `examples`, `context`, and `readonly` are also stripped, and that `required` and `$ref` are preserved.

## Contribution

Committed to trunk on 2026-06-02 by **@gziolo** (Greg Ziółkowski), with props to **@jorgefilipecosta** (Jorge Filipe Costa). The change is filed under Trac ticket #64955. The diff shows no recorded discussion on alternative approaches beyond the docblock revision from "deny-list" to "allow-list" framing.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

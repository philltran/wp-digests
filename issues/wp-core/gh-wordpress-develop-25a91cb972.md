# HTML API: Refactor `wp_get_admin_notice()`.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Dennis Snell
- **Committed:** 2026-08-28
- **Commit:** [`25a91cb972`](https://github.com/WordPress/wordpress-develop/commit/25a91cb9724e0fbda50a0d281c8f84e34aa738a8)
- **Usefulness:** 3/5

## Summary

`wp_get_admin_notice()` now builds the notice's opening `<div>` with `WP_HTML_Tag_Processor` instead of string concatenation. The `id`, `type`, `additional_classes` and `attributes` arguments are applied through `set_attribute()` and `add_class()`, so they are escaped and can no longer break out of their attribute boundaries. Previously, malicious or malformed values could corrupt the markup, and the output then got mangled when passed through `wp_kses()`. The output for well-formed input is intended to stay the same.

## Impact

- **Plugin & theme developers:** No action is needed for well-formed calls to `wp_admin_notice()` / `wp_get_admin_notice()`. Output for malformed input changes: values that used to inject markup or split attributes are now kept inside a single escaped attribute value.
- **Code relying on the old output for broken input:** If any code or test compared exact strings for unsafe `id`, `type` or `additional_classes` values, it will now see different markup. For example, a `type` of `"><script>...` now ends up escaped inside the `class` attribute instead of being emitted as a live tag.
- **Filter consumers:** The `wp_admin_notice_args` and `wp_admin_notice_markup` filters remain. Code hooking `wp_admin_notice_markup` now receives markup where attribute boundaries are already normalized.
- **Attribute arrays:** Boolean handling differs in detail. Numeric-keyed entries (`$args['attributes'][] = 'disabled'`) are set as boolean attributes named after the value. Named entries set to `true` become boolean attributes, and `false` is skipped. Falsy non-`false` values, such as `''` or `0`, are now emitted (for example `data-x=""`), where the old code skipped them.
- **Site owners / hosting / headless:** No action required.

## Technical details

The change is in `src/wp-includes/functions.php`, `wp_get_admin_notice()`, after the `wp_admin_notice_args` filter runs.

**Before:** the function concatenated `$id`, `$classes` and `$attributes` strings and `sprintf`'d them into `<div %1$sclass="%2$s"%3$s>%4$s</div>`. The `id` and `type` values were not escaped. Only the `attributes` values went through `esc_attr()`.

**After:**

```php
$html_builder = new WP_HTML_Tag_Processor( "<div class=\"notice\">{$wrap_opener}" );
$html_builder->next_token();
// set_attribute( 'id', ... ), add_class( "notice-{$type}" ),
// add_class( 'is-dismissible' ), add_class( $class_name ) per additional class,
// set_attribute( $name, ... ) per attributes entry
$markup  = $html_builder->get_updated_html();
$markup .= $message;
$markup .= "{$wrap_closer}</div>";
```

The processor is seeded with the `<div class="notice">` tag, plus `<p>` when `paragraph_wrap` is not `false`. Attributes are then mutated through its API. `$message` is appended raw after the serialized opening tag.

Other changes:
- The whitespace check for `type` changed from `str_contains( $type, ' ' )` to `strlen( $type ) !== strcspn( $type, " \f\t\r\n" )`, so it now matches all HTML whitespace characters and not just a space.
- `additional_classes` entries are added individually through `add_class()` instead of being imploded.
- In `attributes`, non-string values are cast with `(string)` and trimmed.
- Tests in `wpAdminNotice.php` and `wpGetAdminNotice.php` switch from `assertSame` to `assertEqualHTML`. The expectations for the unsafe `type`, `id` and `additional_classes` cases change to show the payload escaped inside the attribute value (`&quot;`) instead of leaking out as tags.

## Contribution

Developed in PR #13273 and discussed in Trac #65984. The commit credits dmsnell (author), joedolson, johnbillion and jorbin. The record shows no discussion of rejected alternatives. The commit message frames the behavioral changes as affecting only already-broken cases.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

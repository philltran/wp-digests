# REST API: Normalize non-integer attachment `filesize` metadata.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Weston Ruter
- **Committed:** 2026-07-21
- **Commit:** [`128db3cbea`](https://github.com/WordPress/wordpress-develop/commit/128db3cbeaa53869cffde9737dc8e5dbad2333c7)
- **Usefulness:** 4/5

## Summary

The REST API now safely normalizes the `filesize` metadata for attachments, preventing `TypeError` crashes and bogus values when plugins store the field as a string or float. The `get_attachment_filesize()` method validates that the stored value is numeric and greater than zero before casting it to an integer, and the endpoint schema now explicitly allows `null` when the size is unavailable or invalid. This ensures consistent, type-safe responses from the `/wp/v2/media` endpoint.

## Impact

- **Plugin & theme developers:** No immediate code changes required, but any code parsing the `/wp/v2/media` response should update type checks to handle `null` for the `filesize` property.
- **Hosting & platform teams:** Eliminates fatal `TypeError` crashes in REST responses caused by third-party plugins storing non-integer `filesize` values in attachment metadata.
- **Headless & REST consumers:** The `filesize` field in the media endpoint schema now returns `integer | null` instead of strictly `integer`.

## Technical details

The change modifies `WP_REST_Attachments_Controller` in `src/wp-includes/rest-api/endpoints/class-wp-rest-attachments-controller.php`. In `get_item_schema()`, the `filesize` property schema is updated to allow `null`:
```diff
- 'type'        => 'integer',
+ 'type'        => array( 'integer', 'null' ),
```
The `get_attachment_filesize()` method now validates metadata with `is_numeric( $meta['filesize'] ) && $meta['filesize'] > 0` before casting to `(int)`. If the metadata is missing, non-numeric, or zero/negative, the method falls through to calculate the size directly from the file system. A `@phpstan-return non-negative-int|null` annotation was added to the method signature to align static analysis with the new validation logic.

## Contribution

Developed in PR #12611 and committed on 2026-07-21 by Weston Ruter, with props to apermo, xate, and mukesh27. The fix addresses a long-standing type-safety gap in attachment metadata handling, following up on earlier revisions (r62813, r61703). The implementation focused on defensive validation and schema alignment rather than architectural changes, ensuring remote storage plugins and other metadata consumers no longer trigger fatal errors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

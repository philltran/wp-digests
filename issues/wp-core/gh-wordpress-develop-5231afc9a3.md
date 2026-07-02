# REST API: Expose output format and progressive flags on attachments.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Adam Silverstein
- **Committed:** 2026-07-01
- **Commit:** [`5231afc9a3`](https://github.com/WordPress/wordpress-develop/commit/5231afc9a36ece00ca183d70510d4e3a903562d9)
- **Usefulness:** 4/5

## Summary

Two new readonly fields, `image_output_format` and `image_save_progressive`, are now exposed on individual image attachments via the REST API when requested in the `edit` context. This change shifts per-image output formatting and progressive encoding flags from the global REST index (`/wp/v2/`) to media endpoints, allowing developers to inspect the actual output MIME type and interlacing behavior resolved by the `image_editor_output_format` and `image_save_progressive` filters on a per-file basis.

## Impact

- **Plugin & theme developers**: Can now read per-attachment format conversion (`string|null`) and progressive flag (`boolean`) directly from `/wp/v2/media/{id}` responses. Essential for building media optimization dashboards or headless frontend renderers that need to know the final saved format.
- **REST API consumers**: The global index response no longer returns `image_output_formats`, `jpeg_interlaced`, `png_interlaced`, or `gif_interlaced`. This is a breaking change for any client relying on those root-level keys; migrate to per-attachment endpoints if these flags are required.
- **Site owners & standard headless clients**: No action required unless you depend on the removed global REST index properties for image processing logic.

## Technical details

The implementation moves the evaluation of image save filters from `WP_REST_Server::get_index()` into `WP_REST_Attachments_Controller::prepare_item_for_response()`. 
- The new fields are gated by `wp_attachment_is_image()` and restricted to the `edit` context in the schema.
- `image_output_format` evaluates `image_editor_output_format` using the attachment's real filename and MIME type, returning the mapped output MIME or `null` if unchanged.
- `image_save_progressive` applies the corresponding filter against the attachment's MIME type and returns the boolean result.

**Schema registration example:**
```php
$schema['properties']['image_output_format'] = array(
    'description' => __( 'The output MIME type this image should be converted to, based on the image_editor_output_format filter. Null if no conversion is needed.' ),
    'type'         => array( 'string', 'null' ),
    'context'      => array( 'edit' ),
    'readonly'     => true,
);
```
The diff removes the previous loop that calculated `$available['image_output_formats']` and hardcoded interlaced flags from `WP_REST_Server::get_index()`.

## Contribution

Merged by Adam Silverstein on 2026-07-01 (Props westonruter). Resolves #65367. The change aligns with companion Gutenberg updates (#75793, #75784) to expose per-image processing flags directly on media endpoints, moving away from global index calculations that could not accurately reflect per-attachment filter decisions.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

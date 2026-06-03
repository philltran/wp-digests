# #78788: Tests: Temporarily disable REST index output-format assertions pending Core fix

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `No Core Sync Required`
- **Merged:** [`db0cf2f`](https://github.com/WordPress/gutenberg/commit/db0cf2f5f6ffd8be8d328af6de9801a0992ea607)
- **Discussion:** [#78788](https://github.com/WordPress/gutenberg/pull/78788) · 3 comments · 0 reactions

## Summary

Four `assertArrayNotHasKey` assertions in `Media_Processing_Test::test_get_rest_index_should_return_additional_settings_can_upload_files` have been temporarily commented out to restore CI green on the Gutenberg PHP unit test job. The assertions verified that `image_output_formats`, `jpeg_interlaced`, `png_interlaced`, and `gif_interlaced` are **absent** from the REST API root index (`/wp/v2` index response). They are failing because Core changeset 62428 re-introduced client-side media processing by reverting its earlier removal, and that revert restored an older version of `WP_REST_Server::get_index()` that still exposes those keys. The permanent fix — removing those file-less index fields in favor of per-attachment `image_output_format` and `image_save_progressive` response fields — is tracked in `wordpress-develop#12007`.

## Impact

- **Plugin & theme developers / REST consumers:** No behavior change in production code — only test assertions are affected. The REST API root index still exposes `image_output_formats`, `jpeg_interlaced`, `png_interlaced`, and `gif_interlaced` when running against current Core `trunk`; that is the upstream Core state, not a Gutenberg regression.
- **CI / platform teams running Gutenberg's PHPUnit suite against Core trunk:** Tests now pass again. Without this patch, the PHP unit test job was failing on every run against `trunk`.
- **No action required** for production sites or plugin consumers until `wordpress-develop#12007` merges, at which point those index keys will be removed from `WP_REST_Server::get_index()` and the assertions will be re-enabled.

## Technical details

The change is confined to `phpunit/media/media-processing-test.php`. Inside `test_get_rest_index_should_return_additional_settings_can_upload_files`, the four `assertArrayNotHasKey` calls are commented out and replaced with a `TODO` block linking to `wordpress-develop#12007`:

```php
// Before
$this->assertArrayNotHasKey( 'image_output_formats', $data );
$this->assertArrayNotHasKey( 'jpeg_interlaced', $data );
$this->assertArrayNotHasKey( 'png_interlaced', $data );
$this->assertArrayNotHasKey( 'gif_interlaced', $data );

// After (commented out pending Core fix)
// $this->assertArrayNotHasKey( 'image_output_formats', $data );
// $this->assertArrayNotHasKey( 'jpeg_interlaced', $data );
// $this->assertArrayNotHasKey( 'png_interlaced', $data );
// $this->assertArrayNotHasKey( 'gif_interlaced', $data );
```

The remaining assertions (`assertArrayHasKey( 'image_size_threshold' )` and `assertArrayHasKey( 'image_sizes' )`) are untouched. The root cause is that `WP_REST_Server::get_index()` in Core `trunk` (post-revert of the earlier client-side media removal) exposes the four output-format fields globally on the index for users with `upload_files` capability. Gutenberg PR #75793 had previously migrated these to per-attachment `image_output_format` and `image_save_progressive` fields on the attachment response schema; `wordpress-develop#12007` will complete that migration in Core itself.

## Contribution

Opened and merged by @adamsilverstein. @alecgeatches is credited as a co-author. @cbravobernal cherry-picked the commit to the `release/23.3` branch (`086b2c9c65b`) to include it in the next release. The only design discussion was a brief exchange where @adamsilverstein offered to delete the tests outright rather than comment them out; the comment-out-with-TODO approach was kept as it preserves the intent and provides a clear reactivation path once the upstream Core PR lands.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

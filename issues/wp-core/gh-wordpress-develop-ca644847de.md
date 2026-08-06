# HTML API: Respect enqueued updates in `get_attribute_names_with_prefix()`.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Dennis Snell
- **Committed:** 2026-07-31
- **Commit:** [`ca644847de`](https://github.com/WordPress/wordpress-develop/commit/ca644847def89cb8159831332377bc9b13fc9f41)
- **Usefulness:** 3/5

## Summary

`WP_HTML_Tag_Processor::get_attribute_names_with_prefix()` now correctly reflects pending attribute and class modifications before `get_updated_html()` is called. Previously, the method only inspected the current `$this->attributes` array, returning stale results that omitted newly added attributes or included ones queued for removal. This fix aligns the method with the processor’s internal update queue, ensuring immediate consistency after calls like `set_attribute()`, `remove_attribute()`, or `add_class()`.

## Impact

- **Plugin & theme developers**: Will now receive accurate attribute lists immediately after modifying tags, without needing to serialize the HTML first.
- **Hosting & platform teams**: No configuration or migration changes required.
- **Breaking changes**: None. Existing code continues to work; behavior is simply corrected to match developer expectations.

## Technical details

Modifies `WP_HTML_Tag_Processor::get_attribute_names_with_prefix()` in `src/wp-includes/html-api/class-wp-html-tag-processor.php`. The method previously only iterated `$this->attributes`, ignoring pending changes. The updated logic first checks `$this->classname_updates` to flush class additions/removals via `$this->class_name_updates_to_attributes_updates()`. It then scans `$this->lexical_updates` to populate `$additions` and `$removals` arrays, filtering the base attribute keys against `$removals` and merging `$additions` into the final return value. PHPDoc types for `$classname_updates` and `$lexical_updates` were tightened to `array<non-empty-string, self::ADD_CLASS|self::REMOVE_CLASS>` and `array<int|string, WP_HTML_Text_Replacement>`. Six new PHPUnit tests in `tests/phpunit/tests/html-api/wpHtmlTagProcessor.php` verify immediate consistency with `get_attribute()` after pending updates.

## Contribution

Developed by Dennis Snell as a follow-up to r55203, addressing an oversight documented in Trac ticket #64567. The patch landed via pull request #12757 after review focused on ensuring the method correctly interprets the processor’s internal update queues without triggering a full HTML serialization pass. The author coordinated extensive test coverage to guarantee that mid-stream attribute queries remain consistent with `get_attribute()` and `get_updated_html()`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

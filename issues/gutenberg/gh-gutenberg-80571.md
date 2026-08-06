# #80571: View config: reject shape-mismatched merges, define empty-array semantics, strip nulls from appended members

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `Backported to WP Core`
- **Merged:** [`f69c158`](https://github.com/WordPress/gutenberg/commit/f69c15847fb9f8cbff9fd32b97e0f02640f6e345)
- **Discussion:** [#80571](https://github.com/WordPress/gutenberg/pull/80571) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `Gutenberg_View_Config_Data` merge engine now enforces strict shape matching between patch values and existing configuration, defines explicit semantics for empty arrays, and strips nested `null`s from newly appended list members. Previously, mismatched shapes silently discarded data, empty arrays could wipe associative values, and appended list items retained invalid `null` leaves. These changes prevent silent configuration corruption in the View Config system.

## Impact

- **Plugin & theme developers / block authors:** If you programmatically modify view configuration via `Gutenberg_View_Config_Data::merge()` or `::replace()`, ensure patch arrays match the expected shape (associative arrays merge into associative arrays, lists into lists). Mismatched non-empty patches will now trigger a `_doing_it_wrong()` notice and leave the existing value intact.
- **Platform & hosting teams:** No migration required. The change is backward-compatible in behavior (it preserves existing data rather than overwriting it) but will surface PHP notices for malformed patches.
- **No action required** for sites not programmatically manipulating view configuration.

## Technical details

The diff modifies `lib/compat/wordpress-7.1/class-gutenberg-view-config-data.php`, specifically the `merge_properties()` and `merge_list_by_identity()` methods.
- `merge_properties()` now checks `array_is_list()` on both `$current` and `$incoming`. If a non-empty list is patched over a non-empty associative array (or vice versa), it calls `_doing_it_wrong()` and returns `$current` unchanged.
- Empty arrays (`array() === $incoming`) are explicitly treated as a no-op under `merge()`, preventing them from wiping existing associative values.
- `merge_list_by_identity()` now wraps appended items in `$this->strip_nulls( $item )` instead of pushing them raw, ensuring nested `null` leaves are dropped during identity-aware list merges.
- Docblocks for `merge()` and `replace()` were updated to document these shape rules and empty-array semantics.

Example of the new shape guard in `merge_properties()`:
```php
if ( array() !== $incoming && is_array( $current ) && ! array_is_list( $current ) && array() !== $current ) {
    _doing_it_wrong(
        __METHOD__,
        esc_html__( 'A view configuration patch value must match the shape of the value it patches: a list merges into a list, and an associative array into an associative array.', 'gutenberg' ),
        '7.1.0'
    );
    return $current;
}
```

## Contribution

The PR was opened by @jorgefilipecosta as a follow-up to #80319, with co-authorship from @oandregal. Because it was initially submitted from a fork, automatic cherry-picking failed, requiring manual conflict resolution before @t-hamano successfully backported the commit to `wp/7.1` via #80829. The author noted the defects were identified via AI-assisted code review, with the fix and tests drafted accordingly.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

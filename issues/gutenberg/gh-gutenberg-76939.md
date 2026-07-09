# #76939: RTC: Add filterable flag for meta box RTC compatibility

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Task`, `[Type] Enhancement`, `Needs Dev Note`, `[Package] Edit Post`, `[Feature] Real-time Collaboration`, `Backported to WP Core`
- **Merged:** [`fa4eb5b`](https://github.com/WordPress/gutenberg/commit/fa4eb5b1744268e1bf41201a08450eea14457894)
- **Discussion:** [#76939](https://github.com/WordPress/gutenberg/pull/76939) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Introduces a new `__rtc_compatible_meta_box` argument for `add_meta_box()`, allowing developers to opt legacy meta boxes out of the Real-Time Collaboration (RTC) single-user lockout behavior. Previously, the presence of any classic meta box would trigger a modal warning and force the editor into a single-user mode; this change enables collaboration to remain active for explicitly compatible meta boxes.

## Impact

- **Plugin authors**: Can add `['__rtc_compatible_meta_box' => true]` to the `$args` array in `add_meta_box()` to safely participate in RTC.
- **Site administrators & developers**: Can use the `filter_block_editor_meta_boxes` filter to retroactively mark third-party meta boxes as compatible without waiting for upstream plugin updates.
- **No action required** for sites not using Real-Time Collaboration (Settings > Writing > "Enable real-time collaboration").
- Behavior changes if RTC is enabled and previously-incompatible meta boxes are now marked compatible; ensure testing of concurrent edits.

## Technical details

A callback attached to `filter_block_editor_meta_boxes` inspects the `args` array of each registered meta box for the new `__rtc_compatible_meta_box` flag. Compatible IDs are compiled and passed to the client via an inline script. On the JS side, a new `setRtcCompatibleMetaBoxIds` action in the `core/edit-post` store populates a client-side list of safe IDs. The `useMetaBoxInitialization` hook compares active meta boxes against this list and only triggers the RTC lockout modal if an ID is missing. This follows the established pattern for `__back_compat_meta_box` and `__block_editor_compatible_meta_box`.

## Contribution

Opened by @alecgeatches to resolve issue #76940, which documented that any legacy meta box would unconditionally break multi-user collaboration. The PR was merged with co-authors (@chriszarate, @ingeniumed, @maxschmeling, @sc0ttkclark) and backported to WordPress Core as part of the broader Real-Time Collaboration platform work.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

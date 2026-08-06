# #81206: Media Attached to: Fix issue with the popover unexpectedly flipping, tweak wording

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Feature] Media`, `Backported to WP Core`
- **Merged:** [`6b40482`](https://github.com/WordPress/gutenberg/commit/6b40482d0f4e7007ea1c6dba3d4d0acce63885c7)
- **Discussion:** [#81206](https://github.com/WordPress/gutenberg/pull/81206) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The "Attached to" media field in the block editor now reserves space for its suggestion list to prevent the parent popover from unexpectedly flipping or shrinking when opened. The change also simplifies the field's help text, removes an explicit detach button, and fixes a debouncing bug that caused API requests on every keystroke.

## Impact

- **Plugin & theme developers** using the `@wordpress/media-fields` package will see the `attached_to` control render with a more stable popover height and corrected search behavior.
- **No breaking changes.** The `ComboboxControl` API remains compatible, though `expandOnFocus` is now explicitly `false`.
- **No migration required.** Detaching media still works via the field's reset/X icon; the inline help text simply no longer references a detach button.

## Technical details

In `packages/media-fields/src/attached_to/edit.tsx`, the `onFilterValueChange` handler now uses `useDebounce( useEvent( onValueChange ), 300 )` to correctly debounce search queries and avoid stale closures. `expandOnFocus` is set to `false` on `ComboboxControl` to defer rendering the suggestion list until a search occurs. The explicit detach `Button` and `createInterpolateElement` help text were replaced with a static string, and the `setOptions( [] )` call in `handleDetach` was removed to allow immediate re-attachment.

A new `packages/media-fields/src/attached_to/style.scss` targets `.components-popover__content:has(.dataviews-media-field__attached-to)` to apply a transparent `margin-bottom` matching the expected suggestion list height. This forces `floating-ui` to measure adequate space before positioning, preventing the popover from collapsing or flipping. Once `.components-form-token-field__suggestions-list` appears, the margin collapses to zero and the list height is locked to prevent popover resizing during typing.

## Contribution

Opened and merged by @andrewserong with review from @ramonjd. The PR was backported to the `wp/7.1` branch. Reviewers acknowledged the underlying UX jank but agreed this CSS workaround and debouncing fix were the best short-term solution before a full combobox redesign in 7.2+.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

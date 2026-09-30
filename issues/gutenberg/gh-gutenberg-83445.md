# #83445: Fields: Use the post type labels in the featured image field and drop `mediaUploadProps`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] DataViews`, `[Package] Fields`
- **Merged:** [`4fcf120`](https://github.com/WordPress/gutenberg/commit/4fcf120575118665381e63dd28a5541881df8acc)
- **Discussion:** [#83445](https://github.com/WordPress/gutenberg/pull/83445) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The featured image field in `@wordpress/fields` now derives its button label from the post type's `set_featured_image` label and its DataViews media modal title from the `featured_image` label, matching the classic sidebar panel. The `mediaUploadProps` prop added to `MediaEdit` in #82678 is removed before the package's RC, since the featured image field (its only consumer) now renders the internal `MediaEditControl` directly with explicit `featuredImageFlow` and `pickerTitle` props instead of a generic pass-through object.

## Impact

- **Plugin & theme developers tracking `@wordpress/fields`:** The `mediaUploadProps` prop on `MediaEdit` is gone. The package is not yet published, so no shipped code should reference it. The internal `MediaEditWithFilteredPicker` export is also removed; `MediaEditControl` is now the exported internal component.
- **Plugins extending `editor.MediaUpload` (e.g. Jetpack External Media):** Both `featuredImageFlow` and `unstableFeaturedImageFlow` are still passed to the picker, so existing callbacks that read either prop continue to recognize the featured image flow. No action required.
- **Site owners / CPT authors:** If a registered post type defines custom `featured_image` or `set_featured_image` labels, the DataViews-based featured image field will now display them (e.g. a "Book" CPT with `set_featured_image: 'Set cover'` shows that on the button). The classic sidebar already did this; the new field now matches.
- **No breaking change for published code** — the fields package has not reached RC.

## Technical details

In `packages/fields/src/components/media-edit/index.tsx`, the previously private `MediaEditControl` is now exported, and `MediaEditWithFilteredPicker` is removed. `MediaEdit` is reduced to a one-line wrapper around `MediaEditControl`. The `mediaUploadProps` prop is stripped from `MediaEditProps` in `types.ts` and from the JSDoc/README.

`MediaEditControl` gains three new internal props:

```tsx
isPickerFiltered?: boolean;   // resolve picker through `editor.MediaUpload` filter
featuredImageFlow?: boolean;  // open the featured-image media frame
pickerTitle?: string;         // title shown in the DataViews media modal
```

The picker component is selected at render time:

```tsx
const MediaUploadComponent = isPickerFiltered
  ? FilteredMediaUpload   // withFilters( 'editor.MediaUpload' )( ConditionalMediaUpload )
  : ConditionalMediaUpload;
```

In `packages/fields/src/fields/featured-image/edit.tsx`, the component now calls:

```tsx
const labels = useSelect(
  ( select ) => select( coreStore ).getPostType( data.type )?.labels,
  [ data.type ]
);
```

The button label is overridden via a memoized `field` object whose `placeholder` is set to `labels?.set_featured_image`. The picker title is `labels?.featured_image || field.label`. Both `featuredImageFlow` and `unstableFeaturedImageFlow` are spread onto the `MediaUploadComponent` so that plugins reading either name still detect the featured-image context.

Before (from #82678):
```tsx
const mediaUploadProps = { featuredImageFlow: true, unstableFeaturedImageFlow: true };
<MediaEditWithFilteredPicker { ...props } mediaUploadProps={ mediaUploadProps } />
```

After:
```tsx
<MediaEditControl { ...props } isPickerFiltered featuredImageFlow
  pickerTitle={ labels?.featured_image || field.label } />
```

Bundle impact is negligible: +96 B total, +111 B in `build/scripts/editor/index.min.js`.

## Contribution

Opened by @ntsekouras as a follow-up to a review comment on #82678. Co-authored with @jorgefilipecosta and @Mamaduka. The PR carries only 2 comments and no reactions; the record shows no design debate or rejected alternatives beyond the stated rationale for dropping `mediaUploadProps` (most of its forwarded props were incompatible with one of the two picker implementations).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

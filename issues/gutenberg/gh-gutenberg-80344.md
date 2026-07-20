# #80344: ContentEditableControl: fix invalid label association with contenteditable div

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `[Package] Components`, `Backported to WP Core`, `[Package] DataViews`
- **Merged:** [`d75dbfd`](https://github.com/WordPress/gutenberg/commit/d75dbfd2c82c457ef1dba1573a6424992da3af34)
- **Discussion:** [#80344](https://github.com/WordPress/gutenberg/pull/80344) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

ContentEditableControl previously used a `<label for>` attribute pointing to a `div contenteditable`, which violates the HTML spec and triggers Chrome console errors. This PR replaces the invalid association with `aria-labelledby`, removes a redundant `aria-label`, and manually wires label-click focus behavior using `BaseControl.VisualLabel` and an `onClick` handler.

## Impact

- **Plugin & theme developers:** No action required. The component's public props and rendered output remain functionally identical.
- **Console & accessibility:** Eliminates Chrome's "Incorrect use of `<label for=FORM_ELEMENT>`" console error and corrects the accessible name association for assistive technologies.
- **DataViews/RichText consumers:** Implicitly benefits from the fix in `RichTextControl` and related DataForm controls, though no migration or code changes are needed.

## Technical details

In `packages/components/src/content-editable-control/index.tsx`, the `label` prop is no longer passed to `useBaseControlProps`/`BaseControl`. Instead, the label is rendered explicitly as `<BaseControl.VisualLabel id={ labelId } onClick={ () => editableRef.current?.focus() }>` (or `<VisuallyHidden>` when `hideLabelFromVision` is true), where `labelId` is derived as `${ controlProps.id }__label`. The editable `div` receives `aria-labelledby={ labelId }` instead of `aria-label={ label }`. `useMergeRefs` and `useRef` are used to attach the internal ref to the `div` without breaking the forwarded ref pattern. Test assertions in `packages/components/src/content-editable-control/test/index.tsx` and `packages/dataviews/src/components/dataform-controls/richtext/test/control.tsx` were updated to verify the new `aria-labelledby` association and label-click focus behavior.

## Contribution

Opened by @adamsilverstein and merged in commit `d75dbfd`. Co-authored by @ciampo, @Mamaduka, and @mirka. The change was manually backported to the `wp/7.1` branch via PR #80441 after an initial automatic cherry-pick encountered a conflict.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

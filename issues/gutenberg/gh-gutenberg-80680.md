# #80680: Components: Remove ValidatedTextControl private API

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`, `[Package] Format library`, `[Package] DataViews`
- **Merged:** [`5abfd41`](https://github.com/WordPress/gutenberg/commit/5abfd415e84917f6787d5d92b034fa0e891ae365)
- **Discussion:** [#80680](https://github.com/WordPress/gutenberg/pull/80680) · 5 comments · 0 reactions
- **Usefulness:** 2/5

## Summary

Gutenberg removes the private `ValidatedTextControl` component from `@wordpress/components`, along with its tests and Storybook story. It duplicated `ValidatedInputControl`, which is what the codebase actually uses. The two remaining internal call sites (the DataForm validation story and the `core/math` format popover) now use `ValidatedInputControl`.

## Impact

- **Plugin & theme developers:** Anyone reaching `ValidatedTextControl` through `unlock( privateApis )` from `@wordpress/components` will get `undefined` after this lands. Switch to `ValidatedInputControl`. The PR author reports no uses in related product repos.
- **Core/Gutenberg contributors:** Internal call sites in `@wordpress/dataviews` and `@wordpress/format-library` are already updated.
- **Site owners / hosting / REST consumers:** No action required.
- The change is labelled `[Type] Breaking Change` and recorded in the `@wordpress/components` CHANGELOG under Breaking Changes, but it only affects code that opted into private APIs.

## Technical details

The diff does the following:

- Deletes `validated-form-controls/components/text-control.tsx`, its test (`test/text-control.tsx`) and story (`stories/text-control.story.tsx`).
- Removes the `export * from './text-control'` line from `validated-form-controls/components/index.ts`.
- Drops `ValidatedTextControl` from the import and `lock( privateApis, { ... } )` call in `packages/components/src/private-apis.ts`.
- Narrows a selector in `validated-form-controls/style.scss` from `:is(textarea, input[type="text"]):invalid[data-validity-visible]` to `textarea:invalid[data-validity-visible]`, with the comment now reading "For TextareaControl". Invalid-state styling for `input[type="text"]` no longer comes from this rule.
- Updates `packages/dataviews/src/dataform/stories/validation.tsx` and `packages/format-library/src/math/index.js` to destructure and render `ValidatedInputControl`.

```js
// before
const { ValidatedTextControl } = unlock( componentsPrivateApis );
// after
const { ValidatedInputControl } = unlock( componentsPrivateApis );
```

The removed component wrapped `TextControl` in `ControlWithError`. The compressed-size bot reports about -96 B on `components/index.min.js`.

## Contribution

During review, @ciampo noted that in the math format popover the input only validates after its first blur, and clicking outside dismisses the popover without showing an error or inserting any math text. He said this is not blocking since trunk behaves the same, and suggested a follow-up. @mirka replied that the popover seems to implement only half of the "validate on popover close" pattern, because it does not block closing on error. She suggested moving to either a modal-style pattern (cancel still allowed) or a pattern that prevents closing on error. That follow-up was not part of this PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #80356: SearchControl: Render the suffix only if there's one.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @juanfra
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`68c652b`](https://github.com/WordPress/gutenberg/commit/68c652bf78638505b6d2f703151695c0db67778e)
- **Discussion:** [#80356](https://github.com/WordPress/gutenberg/pull/80356) · 8 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The `SearchControl` component in `@wordpress/components` now conditionally passes its suffix to `InputControl` only when a reset or close button is actually present. Previously, the component always rendered a `<SuffixItem />` element, which caused the underlying input to retain suffix padding even when the button was hidden. This fix ensures the input’s right padding dynamically adapts to the presence of the suffix, resolving an unbalanced spacing issue with long placeholders.

## Impact

- **Block & plugin developers:** No code changes required. The `SearchControl` component now behaves correctly in the block editor and custom admin UIs.
- **Gutenberg/block editor users:** Fixes a visual padding bug where the search input retained extra right padding when no reset/close button was visible.
- **Hosting & platform teams:** No action required. This is a client-side component fix with no server or database impact.
- **Breaking changes:** None. The public API and props remain unchanged.

## Technical details

The change modifies `packages/components/src/search-control/index.tsx` in the `UnforwardedSearchControl` function. A new boolean `hasSuffix` is computed as `!! onClose || !! value`. The `suffix` prop passed to `StyledInputControl` is now conditionally rendered:

```tsx
// Before
suffix={
    <SuffixItem
        searchRef={ searchRef }
        value={ value }
        onChange={ onChange }
        onClose={ onClose }
    />
}

// After
suffix={
    hasSuffix && (
        <SuffixItem
            searchRef={ searchRef }
            value={ value }
            onChange={ onChange }
            onClose={ onClose }
        />
    )
}
```

`InputControl` uses the truthiness of the `suffix` prop to toggle its internal padding styles. By passing `undefined`/`false` when `hasSuffix` is false, the padding correctly collapses. Unit tests in `packages/components/src/search-control/test/index.tsx` were updated to assert that `getComputedStyle( searchInput ).paddingInlineEnd` changes when the reset button appears and disappears.

## Contribution

Opened by @juanfra as a follow-up to #80315, the PR was reviewed and merged by @ciampo. During review, @ciampo noted that existing tests didn’t catch the padding regression because `SuffixItem` returned `null` internally, and suggested adding explicit padding assertions. @juanfra updated the test suite to verify `paddingInlineEnd` behavior. A discussion about migrating `InputControl`’s horizontal paddings to `padding-inline` was deferred to a separate PR to avoid scope creep.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

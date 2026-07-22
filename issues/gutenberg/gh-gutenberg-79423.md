# #79423: Audio: Apply `inert` directly instead of wrapping in `Disabled`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Audio`
- **Merged:** [`0866d6e`](https://github.com/WordPress/gutenberg/commit/0866d6e4a9de2c8bf9f1f14e5fc595d759c0acf4)
- **Discussion:** [#79423](https://github.com/WordPress/gutenberg/pull/79423) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Audio block now applies the `inert` attribute directly to the `<audio>` element in the editor instead of wrapping it in the `Disabled` component. This eliminates an unnecessary wrapper node, aligning the editor markup with the frontend `save` output and removing an unused dependency. Deselected audio blocks remain unresponsive to clicks and keyboard focus without altering the saved HTML structure.

## Impact

- **Plugin & theme developers:** No action required. The saved HTML output remains identical, and no public block attributes or APIs were changed.
- **Block developers:** If you are extending the Audio block or relying on a wrapper element around the `<audio>` tag in the editor, note that the DOM structure now matches the frontend exactly (`<figure>` → `<audio>`).
- **Site owners & editors:** No visible change; deselected audio blocks will continue to ignore interaction as expected.

## Technical details

The change modifies `packages/block-library/src/audio/edit.js`. The `Disabled` component import and wrapper are removed. Instead, the `<audio>` element receives an `inert` prop that evaluates to the string `'true'` when `!isSingleSelected`, and `undefined` otherwise.

Before:
```jsx
<Disabled isDisabled={ ! isSingleSelected }>
    <audio controls="controls" src={ src ?? temporaryURL } />
</Disabled>
```

After:
```jsx
<audio
    controls="controls"
    inert={ ! isSingleSelected ? 'true' : undefined }
    src={ src ?? temporaryURL }
/>
```

This directly mirrors the approach applied to the Video block in #79371. The `Disabled` component's context and styles are no longer consumed for this block, reducing editor bundle overhead by 7 bytes.

## Contribution

The change was proposed to align the Audio block with the Video block’s recent `inert` implementation. Review focused on verifying DOM parity and bundle size reduction, with no alternative approaches debated or rejected before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

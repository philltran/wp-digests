# #79371: Video: Apply `inert` directly instead of wrapping in `Disabled`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Video`
- **Merged:** [`9308056`](https://github.com/WordPress/gutenberg/commit/930805671d2b2c0d5fc6693514a8ebb9a3af7a4d)
- **Discussion:** [#79371](https://github.com/WordPress/gutenberg/pull/79371) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Video block's editor component now applies the native `inert` attribute directly to the `<video>` element instead of wrapping it in the `Disabled` React component. This eliminates an unnecessary wrapper node, aligning the editor markup with the frontend `save` output and removing an unused dependency. The change preserves the existing behavior where deselected videos cannot be played or focused in the block editor.

## Impact

- **Block & theme developers**: No breaking changes or migration required. The `Disabled` component remains available for other use cases, but the Video block no longer relies on it.
- **Editors**: No visible change in behavior; deselected videos remain unplayable and unfocusable when block controls are enabled.
- **Platform & hosting teams**: Negligible impact; reduces the editor bundle size by approximately 5 bytes.

## Technical details

In `packages/block-library/src/video/edit.js`, the `Disabled` import was removed and replaced with a direct `inert` attribute on the `<video>` element. The logic evaluates `! isSingleSelected` to conditionally set `inert="true"` or `undefined`.

Before:
```jsx
<Disabled isDisabled={ ! isSingleSelected }>
    <video controls={ controls } poster={ poster } src={ src || temporaryURL } ref={ videoPlayer }>
        <Tracks tracks={ tracks } />
    </video>
</Disabled>
```

After:
```jsx
<video
    controls={ controls }
    inert={ ! isSingleSelected ? 'true' : undefined }
    poster={ poster }
    src={ src || temporaryURL }
    ref={ videoPlayer }
>
    <Tracks tracks={ tracks } />
</video>
```

This change removes the `Disabled` wrapper from the rendered DOM, ensuring the editor's `<figure>` contains only the `<video>` element, matching the frontend output.

## Contribution

The record carries minimal discussion detail; the change was merged as a straightforward refactor after confirming that the native `inert` attribute reliably replaces the `Disabled` wrapper without altering interaction behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

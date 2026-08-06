# #80774: [Backport to 7.1] Dynamic Gallery: Rename toolbar button to Detach and add a modal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Gallery`
- **Merged:** [`8c39b03`](https://github.com/WordPress/gutenberg/commit/8c39b0356bd2ebf36f48cf6102821b87258d26fc)
- **Discussion:** [#80774](https://github.com/WordPress/gutenberg/pull/80774) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Dynamic Gallery block variation now requires explicit confirmation before detaching from its source. The toolbar and inspector panel actions were renamed from “Convert to images” to “Detach” and “Detach Gallery”, and both now trigger a `ConfirmDialog` modal explaining that manual editing will be enabled but automatic updates will stop. This prevents accidental loss of the dynamic link and aligns the detachment UX with the one-way conversion dialog used when entering dynamic mode.

## Impact

- **Block & theme developers**: No code changes required. The block’s internal UI behavior changed, but the `convertToStatic` callback and block structure remain identical.
- **Site owners & editors**: The detachment action now shows a confirmation modal before materializing static inner blocks.
- **No breaking changes or migration steps required.**

## Technical details

The diff modifies `packages/block-library/src/gallery/dynamic-gallery.js`. It introduces a `DetachGalleryDialog` component wrapping `ConfirmDialog` and adds `isConfirmingDetach` state to both `GallerySourcePanel` and `GalleryDynamicView`. The toolbar `ToolbarButton` and inspector `Button` now set this state to `true` on click instead of directly calling `convertToStatic`. The modal’s `onConfirm` callback invokes `convertToStatic()` and resets the state. Button labels were updated to `__( 'Detach' )` and `__( 'Detach Gallery' )`.

**Before (toolbar):**
```jsx
<ToolbarButton onClick={ convertToStatic }>
    { __( 'Convert to images' ) }
</ToolbarButton>
```

**After (toolbar):**
```jsx
<ToolbarButton onClick={ () => setIsConfirmingDetach( true ) }>
    { __( 'Detach' ) }
</ToolbarButton>
{ isConfirmingDetach && (
    <DetachGalleryDialog
        onConfirm={ () => { convertToStatic(); setIsConfirmingDetach( false ); } }
        onCancel={ () => setIsConfirmingDetach( false ) }
    />
) }
```

## Contribution

Opened as a manual backport to the `wp/7.1` branch to resolve changelog merge conflicts from the original #80727. @ramonjd provided a quick review and approved the port. The change was merged with co-authors @mamaduka and @ramonjd. No significant design debates were recorded; it directly ported the confirmed UX pattern from the main branch without modification.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

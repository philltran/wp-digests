# #80847: Page List: Rename the Edit action to Detach and confirm it in a dialog

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Page List`
- **Merged:** [`96c381f`](https://github.com/WordPress/gutenberg/commit/96c381f636a3139095b4d205c374ab44adb51bca)
- **Discussion:** [#80847](https://github.com/WordPress/gutenberg/pull/80847) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Page List block's conversion action has been renamed from "Edit" to "Detach" and now requires explicit confirmation via a `ConfirmDialog`. This change aligns the Page List behavior with the Gallery and Table of Contents blocks, making it clear that detaching permanently converts the dynamic list into static navigation links and stops automatic page updates.

## Impact

- **Block & Theme Developers:** No code changes required. The underlying `convertToNavigationLinks` function and block attributes remain unchanged.
- **Site Owners/Editors:** The toolbar and Settings panel now show "Detach" instead of "Edit". Clicking it opens a confirmation dialog before converting the block, preventing accidental loss of dynamic updates.
- **E2E Test Authors:** Test selectors for the Page List conversion flow must be updated from `Edit`/`Edit Page List` to `Detach`/`Detach Page List`.

## Technical details

The diff replaces the bespoke `ConvertToLinksModal` component (and its `convert-to-links-modal.js` file) with the `__experimentalConfirmDialog` component from `@wordpress/components`. In `packages/block-library/src/page-list/edit.js`, state management shifts from `isOpen`/`openModal`/`closeModal` to a single `isConfirmingDetach` boolean. Both the block toolbar `ToolbarButton` and the Settings panel `Button` now trigger `setIsConfirmingDetach( true )`, which renders the new `DetachPageListDialog` wrapper. The dialog uses `ConfirmDialog` with `confirmButtonText={ __( 'Detach' ) }` and passes `isBusy={ ! hasResolvedPages }` to prevent conversion while entity records are loading. Associated modal styles in `packages/block-library/src/page-list/editor.scss` were removed, and e2e tests in `navigation.spec.js` were updated to match the new UI labels.

## Contribution

Opened by @Mamaduka and merged after brief review. The PR aligns the Page List conversion flow with the recently merged Gallery block detach pattern. During review, @jasmussen noted that "Edit" was previously chosen intentionally, prompting a quick consensus to adopt "Detach" for clarity. @andrewserong raised a minor testing question regarding drag-and-drop prompts, which was clarified as working via the list view sidebar. The change shipped with no major design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

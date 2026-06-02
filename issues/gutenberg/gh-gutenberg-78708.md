# #78708: Media Editor Modal: Try placing the save and cancel buttons in the footer

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Feature] Media`, `[Type] Experimental`
- **Merged:** [`46f6176`](https://github.com/WordPress/gutenberg/commit/46f6176dd6833067ea439ffe48e06fe1a5f683dd)
- **Discussion:** [#78708](https://github.com/WordPress/gutenberg/pull/78708) · 10 comments · 0 reactions

## Summary

An experimental exploration of the new block-editor Media Editor modal (the image crop/edit UI opened from the Image block's Crop toolbar button). The PR relocates the **Save** and **Cancel** action buttons out of the modal's top-right header and into the modal footer, placing them on the same row as the existing editing controls (undo/redo and the crop/rotate/flip toolbar). The goal is to make the editor feel like a conventional WP modal — where confirm/dismiss actions live in the footer — rather than putting action buttons at the top-right. It is explicitly a work-in-progress to drive discussion about control placement, not a finalized design.

## Impact

- **Plugin & theme developers:** No action required. This targets an experimental, in-development Media Editor modal (labels `[Feature] Media`, `[Type] Experimental`) and is not a stable/public API. Don't build against the button placement or modal layout yet.
- **Site owners / editors:** No action required. If/when shipped, the only user-facing change is where the Save and Cancel buttons appear in the image-crop modal (footer instead of top-right). There is no change to crop/rotate/flip functionality.
- **Headless & REST consumers:** Not affected — this is purely an editor UI layout change.

## Technical details

This is a layout/placement change within the new Media Editor modal reached via the Image block's **Crop** toolbar control. The Save and Cancel buttons are moved from the modal header into the footer, grouped with undo/redo and the crop-action toolbar (snap-rotate / flip).

Responsive behavior described in the PR:
- **Desktop:** undo/redo + toolbar controls + Save/Cancel share a single footer row.
- **Mobile / collapsed:** the controls reflow into stacked rows (top: fine-grained rotation; middle: crop-action toolbar buttons; bottom: undo/redo/reset + Cancel/Save).

The only concrete measured change in the provided material is bundle size: `+416 B` total, with edits to `build/scripts/editor/index.min.js` (+193 B) and the editor stylesheets `build/styles/editor/style.css` / `style-rtl.css` (~+64 B each). No new hooks, filters, `block.json` fields, REST schema, or DB changes are introduced. Placement of other elements (e.g. the keyboard-shortcuts button) is explicitly left out of scope. The author notes the implementation itself is throwaway ("please ignore the code for now"), so no stable usage pattern is established.

## Contribution

Opened by **@andrewserong** as part of the Media Editor work tracked in issue #73771, and merged in `46f6176`. The author framed it as an open question rather than a finished proposal, asking reviewers where the controls should live across breakpoints. **@ramonjd** gave the main design feedback, arguing for thumb ergonomics on mobile: the most-repeated, tactile manipulation controls (rotation ruler, rotate, flip) should sit at the very bottom closest to the hand, with the less-frequent document/modal actions (Save/Cancel) placed slightly above. AI tooling (Claude Code) was used in authoring the PR.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #80689: DataForm: Make the panel layout edit button the real field trigger

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Package] DataViews`
- **Merged:** [`1e40a0d`](https://github.com/WordPress/gutenberg/commit/1e40a0d4a07a7b30730801e5695df536671935d5)
- **Discussion:** [#80689](https://github.com/WordPress/gutenberg/pull/80689) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

In the DataForm `panel` layout, the pencil edit button is now the real click handler and dropdown toggle for a field row, instead of the row `div` forwarding clicks to it. Previously, opening a field by clicking the row and then dismissing the flyout left focus on `div.components-dropdown[tabindex="-1"]`, so Enter did nothing and tabbing resumed from an unexpected place. The button's hit area is stretched over the whole row with `::after`, so whole-row clicking still works and focus now returns to the button.

## Impact

- **Plugin/theme developers using `@wordpress/dataviews` DataForm (`panel` layout):** No API changes. Keyboard and screen-reader behavior improves: one tab stop per row, and focus returns to the edit button after the flyout closes.
- **Custom CSS or tests targeting the panel row:** Code that relied on the row `div` receiving clicks, or on the removed `.dataforms-layouts-panel__summary-button:empty` rule (`min-width` of the admin sidebar width), may need updating. Tests that click the row should still work through the stretched hit area.
- **Users:** Text in a panel row is no longer selectable by drag, since the button overlay sits on top. This trade-off was accepted in the linked issue. Double-clicking a row now toggles the flyout open and closed.
- **Visual side effect:** The edit tooltip now also shows when hovering anywhere on the row. It can be reverted with `showTooltip={ false }` if design prefers.
- **Custom summary renderers:** Summary regions containing links or buttons are layered above the overlay and keep their own pointer and keyboard behavior.
- No migration required.

## Technical details

Changes are in `packages/dataviews/src/components/dataform-layouts/panel/`:

- **`summary-button.tsx`**: The row `div` no longer has `onClick`/`onKeyDown`. The `rowRef`, `editButtonRef`, `handleRowClick` (with its drag-to-select guard via `getSelection()`) and `handleKeyDown` are removed. The pencil `Button` now receives `onClick={ onClick }`. Its `aria-describedby` is `controlId` normally, or `` `${ controlId } ${ errorId }` `` when a validation error is shown.
- **`style.scss`**: `.dataforms-layouts-panel__field-trigger-icon::after { content: ""; position: absolute; inset: 0; }` stretches the hit area over the row. Summary elements matching `:has(a, button)` get `z-index: 1` to stay above it. `.dataforms-layouts-panel__field-label-error-content` drops `position: relative`, `z-index: 1` and `cursor: help` and gets `pointer-events: none`. The `.dataforms-layouts-panel__summary-button:empty` rule and the `vars` import are removed.
- **Error indicator**: `utils/get-label-content.tsx` is deleted and replaced by a new `field-label-content.tsx` component (`FieldLabelContent`). It no longer wraps the label in `Tooltip.Root`. It renders the icon plus a `VisuallyHidden` element with `id={ errorId }` containing the error message. The `labelPosition === 'none'` branch drops the tooltip and `role="img"` span and uses the same component.

Tests and stories added: unit tests in `dataform/test/dataform.tsx` asserting that the invalid-field button has an accessible description containing the error message (with and without labels), a new `ValidationPanelErrorIndicator` story (`stories/validation-panel.tsx`), and three e2e tests in `test/e2e/specs/site-editor/page-list.spec.js` covering row click, focus return after Escape then Enter, and double-click behavior.

The CHANGELOG lists this under both a bug fix and an internal simplification.

## Contribution

The approach restores the structure originally shipped in #75290, which #75565 had replaced to keep row text selectable; that trade-off was accepted in issue #79427. During review, @ciampo noted that the PR's `onErrorClick` path partially recreated the click delegation the issue aimed to remove, since the error label intercepted row clicks and forwarded them to the `Button`. He pushed a follow-up making the edit button the only interactive element, which removed the tooltip click workaround and associated the error message via `aria-describedby`. @jorgefilipecosta agreed, and the PR noted that #78028 may need a rebase because it lists this refactor as a follow-up.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

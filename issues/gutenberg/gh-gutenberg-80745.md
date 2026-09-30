# #80745: Block editor: disclose the nested block count of a multi selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Bug`, `[Feature] Block Multi Selection`, `[Package] Block editor`
- **Merged:** [`4d914a0`](https://github.com/WordPress/gutenberg/commit/4d914a0bf569a8cb9f32ceb58d7de7ffbef26285)
- **Discussion:** [#80745](https://github.com/WordPress/gutenberg/pull/80745) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

When a multi-selection in the block editor includes blocks with nested content, the UI now discloses the total block count instead of only the top-level count. The screen-reader announcement becomes "2 blocks selected, 4 including nested blocks.", and the sidebar multi-selection card shows "2 Blocks" with a secondary line "4 including nested blocks". Selections with no nested blocks are reported exactly as before. This fixes #56068, where selecting a Group and a Paragraph reported only "2 blocks selected".

## Impact

- **Editors / site owners:** Multi-selections containing Groups, Lists, Columns, etc. now show and announce the true number of affected blocks. Screen-reader users get the extra count via the `assertive` live region.
- **Plugin & theme developers:** No API changes. `getSelectedBlockCount` still returns the top-level count, so existing consumers are unaffected.
- **Testing / QA teams:** Any e2e or unit test asserting the exact string `"N blocks selected."` after a multi-selection with nested blocks will now see the longer string. The PR itself updates `list.spec.js` for this reason.
- **Translators:** New strings were added: `'%1$s block selected, %2$s including nested blocks.'` (plural forms) and `'%d including nested block(s)'`.
- **Styling overrides:** The card markup changed (see below); custom CSS targeting `.block-editor-multi-selection-inspector__card` internals may be affected.

No action required for most sites.

## Technical details

**`store/actions.js` (`multiSelect` thunk):** after dispatching, it now computes `nestedBlockCount = select.getClientIdsOfDescendants( select.getMultiSelectedBlockClientIds() ).length`. If non-zero, `speak()` gets a message using `_n()` with positional placeholders: `'%1$s block(s) selected, %2$s including nested blocks.'` where `%2$s` is `blockCount + nestedBlockCount`. Otherwise the original `'%s block(s) selected.'` string is used. Priority remains `'assertive'`.

**`components/multi-selection-inspector/index.js`:**
- `useSelect` now returns `{ selectedBlockCount, totalBlockCount }`, where the total is `getSelectedBlockCount()` plus `getClientIdsOfDescendants( getMultiSelectedBlockClientIds() ).length`.
- The layout swaps the experimental `HStack` from `@wordpress/components` for `Stack` from `@wordpress/ui` (`direction="row"`, `align="center"`, `gap="sm"`), with an inner column `Stack`.
- A new `.block-editor-multi-selection-inspector__card-description` div renders `'%d including nested block(s)'` only when `totalBlockCount > selectedBlockCount`.
- `style.scss` adds the description rule with `color: $gray-700` and imports `@wordpress/base-styles/colors`.
- The now-unneeded `@wordpress/use-recommended-components` suppression for this file is removed from `tools/eslint/suppressions.json`.

**Tests:** the `multiSelect` unit test mock gains `getMultiSelectedBlockClientIds` and `getClientIdsOfDescendants`. A new e2e test in `multi-block-selection.spec.js` (Group with 2 paragraphs + Paragraph, select all twice) asserts both the live-region text and the card text. The existing list e2e expectation changes to `'2 blocks selected, 4 including nested blocks.'`. A CHANGELOG entry is added for `@wordpress/block-editor`.

## Contribution

Authored by @ellatrix with Claude Code assistance (disclosed in the PR), reviewed and merged. The PR description records a rejected alternative: replacing the count with the nested-inclusive sum. That would change the meaning of the public `getSelectedBlockCount` selector and produce confusing numbers in other cases (e.g. a list plus a paragraph announcing "6 blocks selected"), so the top-level count stays primary and the total is added as secondary information. The one CI flag was a known flaky test (#39607) in `writing-flow.spec.js`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

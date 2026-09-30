# #82600: Block alignment: List unavailable wide and full alignments instead of hiding them

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Site Editor`
- **Merged:** [`cd62e1a`](https://github.com/WordPress/gutenberg/commit/cd62e1ae4d97a44347471e29f27de2beb0501857)
- **Discussion:** [#82600](https://github.com/WordPress/gutenberg/pull/82600) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block toolbar's alignment dropdown now lists Wide width and Full width as disabled entries labelled "Not available" when a parent layout withholds them but the theme itself offers them. Previously those options were silently removed from the menu. Blocks that support only wide and full (Group, Columns) now keep an alignment control inside flow layouts that offer neither, where the control used to disappear entirely.

## Impact

**Site owners / editors**
- Inside a container with "Inner blocks use content width" off (e.g. a Group or a Content block), the alignment menu shows `Wide width` and `Full width` as unavailable instead of omitting them. Choosing one does nothing and the menu stays open.
- Group and Columns nested in such a flow layout now show an alignment toolbar button (previously absent). The PR calls this out as a visible change to when the control appears.

**Plugin & theme developers**
- No API removals or deprecations, and no action required.
- Themes that offer no wide size, or no layout at all, see no "unavailable" entries for the alignments they don't offer; those stay hidden because the theme is treated as curating its own options.
- Classic themes without layout support (`supportsLayout: false`) are unchanged: wide/full are either offered (via `alignWide`) or left out.
- Any e2e or unit tests that assert the exact contents of the alignment menu, or the absence of the control on Group/Columns in flow layouts, may need updating.

**Unchanged**
- Blocks inside Flex (Row) or Grid parents still show no alignment control.
- Blocks that don't support wide/full (e.g. Separator) gain no unavailable entries.
- When wide/full are enabled, they behave as before, including the "Max 600px wide" descriptions.

## Technical details

**`use-available-alignments.js`**
- The store selection is extracted into `useAlignmentSettings( isNoneOnly )`, which returns `{ wideControlsEnabled, themeSupportsLayout, isBlockBasedTheme }`. The default export `useAvailableAlignments` is otherwise unchanged in behavior.
- The pure logic moves into a store-free `getAvailableAlignments( controls, layout, settings )`, so rules can be evaluated against several layouts with a single store subscription. This followed review feedback to reduce three subscriptions to one.
- New export `useAlignmentMenu( controls )` returns `{ enabled, unavailable }`. An alignment is `unavailable` only if it is `wide` or `full`, the block supports it, the theme offers it at the root, and the parent layout doesn't.
  - "Theme offers it" is computed by evaluating `DEFAULT_CONTROLS` against `{ ...globalLayout, type: 'constrained' }`, where `globalLayout` comes from `useSettings( 'layout' )`.
  - If the parent layout offers no alignments at all (Flex/Grid), `unavailable` is empty.

**`ui.jsx` (`BlockAlignmentUI`)**
- Switches from `useAvailableAlignments` to `useAlignmentMenu`. It returns `null` only when both `enabled` and `unavailable` are empty.
- If nothing is enabled, it synthesizes `{ name: 'none' }` as an anchor.
- Unavailable entries are spliced in directly after `none` with `isUnavailable: true`. Each gets `info={ __( 'Not available' ) }` and is passed `disabled={ isUnavailable }` to `MenuItem`.
- A saved unavailable value can still render as selected, signalling that the setting exists but has no effect there.

**`hooks/align.jsx`**
- `BlockEditAlignmentToolbarControlsPure` now uses `useAlignmentMenu( blockAllowedAlignments )` so the toolbar control renders whenever there is something to report. The remainder of that hunk is truncated in the provided diff.

**Tests and changelog**
- New jsdom tests cover:
  - the unavailable listing and its ordering
  - non-selectable behavior
  - the unchanged case where every alignment is offered
  - Group-like blocks with `controls = [ 'wide', 'full' ]`
  - Flex parents rendering nothing
  - themes with no layout or no `wideSize`
  - classic themes via `alignWide`
- `CHANGELOG.md` for `@wordpress/block-editor` gets an Enhancements entry.

**Accessibility note:** the PR description says unavailable entries use `aria-disabled` rather than `disabled` so they stay focusable and their explanation is reachable by keyboard and screen reader. The tests assert `aria-disabled="true"` and `toBeEnabled()`. In the visible diff, the code passes `disabled={ isUnavailable }` to `MenuItem`, so `MenuItem`/`Button` is presumably what maps that prop to `aria-disabled`; the mapping is not shown in the diff.

## Contribution

@jasmussen authored the PR with Claude assistance (disclosed in the description). Early review from @andrewserong, @tellthemachines and @jordesign pushed the design toward its final shape: options a theme itself doesn't offer stay hidden, and only options the theme offers but a parent layout withholds are shown as disabled. @andrewserong pushed a commit collapsing three store subscriptions into one, following @tellthemachines' suggestion. A VoiceOver spacing concern about "Not available" was investigated and judged correct behavior, since the label and status are separate flex-column spans, so no extra test was added. The follow-up of explaining *why* an alignment is unavailable and linking to the responsible template or parent block was deferred.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

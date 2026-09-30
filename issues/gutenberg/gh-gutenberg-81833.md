# #81833: Block Styles: Refactor the variation UI to use ui/Button

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `[Package] Block editor`, `[Feature] Block Style Variations`
- **Merged:** [`39088bf`](https://github.com/WordPress/gutenberg/commit/39088bf09ab258fb4e0320b38b8ed446f599283b)
- **Discussion:** [#81833](https://github.com/WordPress/gutenberg/pull/81833) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Block Styles variations picker in the block inspector (`BlockStyles` in `@wordpress/block-editor`) now renders its options with `Button` from `@wordpress/ui` instead of `@wordpress/components`. Labels can wrap onto up to three lines before truncating, where previously they were clamped to one line, so similar style names are easier to tell apart. The variations are also wrapped in a `Composite` radio group, giving arrow-key navigation in all four directions and a single Tab stop for the whole group.

## Impact

**Plugin & theme developers**
- No API change. `BlockStyles` keeps its props (`clientId`, `onSwitch`, `onHoverClassName`), and `registerBlockStyle` usage is unaffected.
- Any custom CSS or tests targeting the old markup may break. The variation options are no longer `button` elements with `aria-current`; they are `role="radio"` items with `aria-checked` inside a `role="radiogroup"`. Selectors that relied on `button.components-button.block-editor-block-styles__item`, the `.is-active` class, or the flex layout of `.block-editor-block-styles__variants` will no longer match.
- E2E tests that locate variations via `getByRole( 'button', { name } )` need to switch to `getByRole( 'radio', { name } )`. The core Playwright spec was updated this way.

**Site owners / editors**
- Long style labels are no longer cut to one line. Keyboard users can move between variations with the arrow keys, and Tab leaves the group.
- Hover and focus still preview a variation, and clicking still applies it.

**Accessibility**
- The picker is now exposed as a labelled radio group (`aria-label` "Styles") instead of a set of buttons.

No migration is required for most sites.

## Technical details

Changes are in `packages/block-editor/src/components/block-styles/index.js` and `style.scss`.

**Component**
- `Button` is now imported from `@wordpress/ui`, with an `eslint-disable-next-line @wordpress/use-recommended-components` comment. `Composite` comes from `@wordpress/components`. `clsx` is dropped.
- The variations are wrapped in `<Composite role="radiogroup" aria-label={ __( 'Styles' ) } focusLoop focusWrap focusShift>`, with `activeId` and `setActiveId` controlled.
- `stylesToRender` is chunked into rows of two (`styleRows`, via `useMemo`), and each row is a `Composite.Row`. Each item is a `Composite.Item` rendered as `<Button tone="neutral" variant={ active ? 'solid' : 'outline' } />`, with `role="radio"` and `aria-checked`.
- Item IDs come from `getCompositeItemId( instanceId, style )`, built from `useInstanceId( BlockStyles, 'block-editor-block-styles' )` and `style.name`.
- `onSetActiveId` maps the composite's active ID back to a style and calls `onSelectStylePreview` when it differs from `activeStyle`. Arrow-key movement therefore also selects the style, as with native radios.
- The `onMouseEnter`/`onFocus`/`onMouseLeave`/`onBlur` handlers still call `styleItemHandler`, and `onClick` still calls `onSelectStylePreview`.
- `Truncate` goes from `numberOfLines={ 1 }` to `3`. The label is `style.label || style.name`.

**Styles**
- `.block-editor-block-styles__variants` changes from flex-wrap to `display: grid`. A new `.block-editor-block-styles__row` uses `grid-template-columns: repeat(2, minmax(0, 1fr))`.
- The custom colour, hover, active and focus-ring rules for `button.components-button.block-editor-block-styles__item` are removed, along with the `word-break: break-all` and `white-space: normal` overrides and the `colors` import. Appearance is now handled by `@wordpress/ui` `Button`.

**Tests**
- `test/e2e/specs/site-editor/block-style-variations.spec.js` switches its locator from `button` to `radio`.
- A CHANGELOG entry is added to `packages/block-editor`.

The compressed-size report shows the bundled block-editor CSS shrinking by roughly 115–150 B and the JS growing by about 210 B.

## Contribution

Authored by @t-hamano to fix issue #40331, with Claude Code (Opus) used to write the code and description. @jasmussen had endorsed this approach on the issue as the simplest fix and a reasonable first step, and repeated that on the PR. @ciampo then suggested merging and iterating rather than continuing review. No competing design was discussed in the PR thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

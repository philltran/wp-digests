# #81373: Global Styles: filter and indicate blocks that have custom styles

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `Global Styles`
- **Merged:** [`d1703e0`](https://github.com/WordPress/gutenberg/commit/d1703e053acc5e32733cd05382a8b9dc61f84bbf)
- **Discussion:** [#81373](https://github.com/WordPress/gutenberg/pull/81373) · 14 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Site Editor's **Styles → Blocks** list now marks every block the user has customized with a small dot, and adds a funnel menu beside the search field to narrow the list to just those blocks. Previously the screen rendered every registered block with nothing distinguishing customized ones, so auditing your own global-styles changes meant drilling into blocks one at a time. A block counts as customized when the *user* layer of the global styles config has a real value under either `styles.blocks[ blockName ]` or `settings.blocks[ blockName ]` — so a block-specific color palette marks a block just as a text color does. The dot is paired with a `VisuallyHidden` "Has custom styles" label, which changes the accessible name of those rows.

## Impact

**Site owners / editors**
- Purely additive UI in Site Editor → Styles → Blocks. No action required.
- The customized filter and search compose, so you can search within the customized subset.

**Plugin & theme developers**
- No public API added or removed. `hasUserStylesForBlock()` is exported from `screen-block-list.tsx` for its unit test, not from the package entry point — don't treat it as stable surface.
- The dot is driven entirely off the *user* global styles layer. Styles a theme ships in `theme.json` will never produce a dot, by design: the merged config would match nearly every block.

**Anyone running E2E tests against the Site Editor**
- **This is the one thing that can break you.** Rows for customized blocks now have an accessible name of `"<Block title> Has custom styles"` rather than just `"<Block title>"`. Any Playwright/Puppeteer selector using an exact accessible name against the Styles → Blocks list will stop matching once that block has user styles. Gutenberg's own `test/e2e/specs/site-editor/push-to-global-styles.spec.js` was updated in this PR from `{ name: 'Heading', exact: true }` to `{ name: /^Heading/ }`; apply the same treatment to your suites.
- New CSS hooks if you assert on markup: `.global-styles-ui-block-types-item__indicator`, `.global-styles-ui-block-types-filter`, `.global-styles-ui-block-types-item__icon`, `.global-styles-ui-block-types-item-list__no-results`.

**New/changed strings for translators:** `Has custom styles`, `Filter blocks`, `All blocks`, `Customized`, `You haven't customized any blocks yet.`

## Technical details

All behavior lives in `packages/global-styles-ui/src/screen-block-list.tsx`, with supporting rules in `src/style.scss`.

**Detection.** Two new functions do the work. `hasAnyValue( value )` walks a value recursively and returns whether any leaf is real — `undefined`/`null` are false, empty arrays are false, but empty strings and `0` are true (an intentionally blank `css` string is a user entry). This matters because clearing a control writes `undefined` in place rather than deleting the key, so `{ color: { text: undefined } }` is an empty husk, not a customization. `hasUserStylesForBlock( user, blockName )` then checks both roots:

```ts
export function hasUserStylesForBlock(
	user: GlobalStylesConfig | undefined,
	blockName: string
): boolean {
	return (
		hasAnyValue( user?.styles?.blocks?.[ blockName ] ) ||
		hasAnyValue( user?.settings?.blocks?.[ blockName ] )
	);
}
```

Because the check recurses, block style *variations* (`styles.blocks['core/quote'].variations.plain`) also count.

**List wiring.** `BlockList` pulls `user` from `GlobalStylesContext` once and builds a `Set` of customized block names in a `useMemo` keyed on `user`, iterating only the keys already present under `styles.blocks` and `settings.blocks` rather than every registered block — explicitly to avoid opening a context subscription per row. Filtering is applied after search: `searchedBlockTypes` → `filteredBlockTypes` when `styleFilter === 'customized'`. `BlockMenuItem` gained an `isCustomized` prop and renders a `VisuallyHidden` label plus an `aria-hidden` `<span>` dot.

**Filter control.** The header becomes an `HStack` wrapping a `size="compact"` `SearchControl` and a `Menu` (from `unlock( componentsPrivateApis )`) triggered by a `funnel`-icon `Button` whose `isPressed` reflects `styleFilter !== 'all'`. The popover holds two `Menu.RadioItem`s under the name `global-styles-block-filter`: `all` and `customized`.

**Empty state and announcements.** A new `EmptyBlockList` distinguishes "You haven't customized any blocks yet." (customized filter on, no search term) from "No blocks found.". The `speak()` result-count effect now also fires on `styleFilter` change and short-circuits the `childElementCount` read to `0` when there are no results — otherwise the empty-state paragraph would be counted as one child and announced as "1 result found."

## Contribution

Opened by @ramonjd to close issue #81316, and layered on (but not blocked by) #81407, which supplies missing styles for the changeset list. The visual treatment churned during review: @andrewserong asked for a compact search field to match the Pages screen, better empty-state copy, and wanted the dot to be clickable or carry a tooltip; @ramonjd then swapped the dot for a per-block changeset list built on the existing `getGlobalStylesChanges()`, and rolled that back to the dot before merge. @jasmussen ruled out a left-border highlight — still the admin focus style — called both the list and dot versions workable, and argued the actual changeset belongs on each block's drilldown (possibly under Advanced) rather than in the list, pulling in @beafialho and the design team for input; reset-from-the-dot and change summaries were deferred to follow-ups. The author disclosed the implementation was written with Claude Code under their review, with design and scope decisions their own.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

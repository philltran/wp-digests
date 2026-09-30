# #81825: Menu: Support multiple item descriptions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`db0c214`](https://github.com/WordPress/gutenberg/commit/db0c21469b2aeeedf1b611d79f50e7a453e38bd1)
- **Discussion:** [#81825](https://github.com/WordPress/gutenberg/pull/81825) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` `Menu` component now accepts multiple direct `Menu.ItemDescription` children after the required `Menu.ItemLabel` on every item variant (`Menu.Item`, `LinkItem`, `CheckboxItem`, `RadioItem`, `SubmenuTrigger`). Previously an item allowed at most one description. Each description is added once to the item's `aria-describedby`, in DOM order, so the accessible description matches the visible content.

## Impact

**Plugin & theme developers using `@wordpress/ui` Menu**
- Additive change: existing items with a single description behave as before. No action required.
- You can now render e.g. a short status plus a longer explanation as separate `Menu.ItemDescription` children.
- Descriptions must still be direct children of the item, immediately following `Menu.ItemLabel`. Wrapping them in a fragment-less custom component or other element still fails validation (in non-production builds, an error is thrown).
- If you pass both `aria-describedby` and a matching `id` on a `Menu.ItemDescription`, the ID now appears only once in the resulting attribute.

**Site owners / hosting / REST consumers:** not affected.

## Technical details

Changes are in `packages/ui/src/menu/`.

- `item.tsx`: `getItemContent()` now destructures `[label, ...descriptions]` and returns `descriptionIds` (array) instead of a single `descriptionId`/`hasDescription`. Validation throws if there is no leading `Menu.ItemLabel` or any remaining child is not `Menu.ItemDescription`. The message is now: `Menu.ItemLabel must be the first direct child of every menu item, followed only by Menu.ItemDescription components.`
- `useItemContent()` resolves each description ID as the consumer-provided `id` or a generated `${useId()}-${index}`. It builds `aria-describedby` from a `Set` of the split consumer `aria-describedby` tokens plus the resolved description IDs, which deduplicates while preserving order. The generated keyboard-shortcut description ID stays last. It also returns a new `contentChildren`, produced by `Children.map` + `cloneElement`, which injects generated IDs only where the child's `id` differs from the resolved one.
- `checkbox-item.tsx`, `link-item.tsx`, `radio-item.tsx`, `submenu-trigger.tsx`, `item.tsx`: each now renders `contentChildren` instead of `children` inside `ItemContent`.
- `context.tsx` / `item-description.tsx`: `descriptionId` removed from `MenuItemContentContextValue`; `ItemDescription` no longer reads the content context and just uses its own `id`.
- `types.ts`: `MenuItemChildren` tuple changes from `[Label, Description | false | null | undefined]` to `[Label, ...(Description | false | null | undefined)[]]`; JSDoc updated on all item prop types.

Because descriptions must be direct children, order and IDs are resolved synchronously during render, with no registration state or layout effects (unlike the arbitrary-descendant approach in #81227 for CollapsibleCard).

```tsx
<Menu.Item>
  <Menu.ItemLabel>Save</Menu.ItemLabel>
  <Menu.ItemDescription>Save to this device.</Menu.ItemDescription>
  <Menu.ItemDescription>Keeps the current version.</Menu.ItemDescription>
</Menu.Item>
```

Tests added in `test/index.jsdom.test.tsx` cover DOM-ordered combination (with external `aria-describedby`, a custom first ID, and a shortcut description last) and deduplication of explicit vs. item description IDs. The `RichItems` story gains a second description, and `packages/ui/CHANGELOG.md` gets an Enhancements entry.

## Contribution

Follow-up to #79560, related to #81227, authored by @ciampo; the description notes it was implemented with AI assistance in Codex and reviewed locally. Discussion on the PR was limited to bot output and a side thread between @jasmussen and @ciampo about icon prefixes and indentation in menus, which @ciampo said is handled through consumer-side composition (e.g. the `Icon` `size`) rather than this change.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

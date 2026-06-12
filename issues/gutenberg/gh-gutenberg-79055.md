# #79055: Update `@ariakit/react` to 0.4.29

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] DataViews`
- **Merged:** [`a73ace3`](https://github.com/WordPress/gutenberg/commit/a73ace34dc9fab22cce4bdeddca6b90cd866e7ff)
- **Discussion:** [#79055](https://github.com/WordPress/gutenberg/pull/79055) · 4 comments · 0 reactions

## Summary

Update `@ariakit/react` to `^0.4.29` across `@wordpress/components` and `@wordpress/dataviews`. This upgrade incorporates an upstream Ariakit fix that moves focus to the newly selected tab upon controlled selection change, eliminating previously necessary manual focus-synchronization workarounds in the `Tabs` component.

## Impact

["- **Plugin & theme developers**: The `Tabs` component's keyboard interaction now aligns with native Ariakit behavior, where programmatic selection changes cause focus to shift to the newly selected tab. This replaces prior custom logic that kept DOM focus attached until explicit arrow-key navigation occurred.", '- **`ToggleGroupControl` users**: Radio inputs now render an auto-generated `name` attribute derived from the parent group ID (e.g., `":r3e:"`). Developers relying on explicit `name` attributes for form grouping or scripting may need to audit their implementations, though functional behavior remains consistent.', '- **Action required**: No code migration needed. Update dependencies and re-run snapshot tests if explicitly asserting DOM structure.']

## Technical details

['- Updates dependency constraints in `packages/components/package.json` and `packages/dataviews/package.json` to `"@ariakit/react": "^0.4.29"`.', "- **Focus sync removal**: Deletes a `useEffect` hook from `packages/components/src/tabs/index.tsx` that previously used `requestAnimationFrame` to force-update Ariakit's store `activeId` when DOM focus drifted away from the selected tab. Aligns imports to remove unused `useEffect`.", '- **Blur handler removal**: Deletes an `onBlur` handler from `packages/components/src/tabs/tablist.tsx` that manually synced `selectedId` to `activeId` when leaving the tablist. Switches imports from a namespace import (`import * as Ariakit`) to named exports (`useStoreState`).', '- **Test alignment**: Updates assertions in `packages/components/src/tabs/test/index.tsx` to expect focus on the newly selected tab rather than retaining focus on the previously focused element.', '- **Snapshot updates**: Refreshes `packages/components/src/toggle-group-control/test/__snapshots__/index.tsx.snap`, adding the new auto-generated `name` attribute to all rendered radio option snapshots.']

## Contribution

Authored by @ciampo and merged with review from @manzoorwanijk and @aduth. The PR closes issue #79051 and extends a previously parked dependency bump (#77954) to adopt the upstream focus fix for controlled `Tabs`. CI validated both unit and e2e suites. The author noted that the behavioral shift addresses a core Ariakit change intended to resolve focus desync issues, making the legacy Gutenberg workarounds obsolete.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

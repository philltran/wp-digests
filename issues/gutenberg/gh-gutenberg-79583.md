# #79583: Tabs: RichText handlers for adding/removing tabs

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Tabs`
- **Merged:** [`4d94e3d`](https://github.com/WordPress/gutenberg/commit/4d94e3d5d648249fa848f73057fe00990997a9d8)
- **Discussion:** [#79583](https://github.com/WordPress/gutenberg/pull/79583) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This merge enhances the core Tabs block editor by wiring up RichText handlers to tab labels, allowing editors to add tabs via Enter key or remove them via backspace. The underlying insert and remove logic was extracted from toolbar controls into a shared React hook to eliminate duplication and prevent unnecessary store subscriptions.

## Impact

['- Block editors / site authors: Improved keyboard workflow for managing tabs without relying on the block toolbar buttons.', '- Plugin & theme developers: No public API changes or breaking updates. Developers extending the Tabs block may reuse the new `useTabActions` pattern for lazy state derivation.', '- Hosting, platform & headless consumers: None. This is strictly client-side editor behavior.']

## Technical details

A new internal hook, `packages/block-library/src/tabs/use-tab-actions.js`, exports a default function that returns `{ insertTab, removeTab }`. The hook uses `useRegistry()` and lazily calls `registry.select( blockEditorStore )` inside the callbacks to avoid automatic re-renders when tab structure changes. In `packages/block-library/src/tab-list/edit.js`, tab labels now pass `__unstableOnSplitAtEnd={ () => insertTab( index + 1 )}` and `onRemove={ () => removeTab( index )}` to the `RichText` component. The `TabToolbarControls` component was moved from `packages/block-library/src/tab-panel/` to `packages/block-library/src/tabs/`; it now consumes `useTabActions()` and retains a lightweight `useSelect` call only for disabling the 'Remove' button when a single tab remains. Import paths in `tab-panel/controls.js`, `tab-panels/edit.js`, and `tabs/edit.js` were updated accordingly.

## Contribution

Opened by @Mamaduka (closes #79536) and merged following review from @t-hamano, @prasadkarmalkar, @jeryj, and @hanneslsm. The PR combined a UX enhancement (wiring RichText handlers for keyboard-based tab management) with a structural refactor (extracting insert/remove logic into `useTabActions` to prevent redundant subscriptions). A minor UI flash when switching tabs was noted in discussion and deferred for later debugging.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #81111: List View: Pass only minimal required data to sub-tree branches

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Block library`, `[Feature] List View`, `[Package] Block editor`, `[Package] Edit Site`
- **Merged:** [`a4694b2`](https://github.com/WordPress/gutenberg/commit/a4694b21b5265b2493d4d0cf1435cade4fe2105a)
- **Discussion:** [#81111](https://github.com/WordPress/gutenberg/pull/81111) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The List View component now passes only the `clientId` string to sub-tree branches instead of the full block object. This eliminates accidental re-renders caused by object reference churn during tree invalidations and prevents auxiliary components like `AdditionalBlockContent` and `NavigationLinkUI` from mounting on every row. The change improves editor performance, particularly in deeply nested block structures and the Navigation block inspector.

## Impact

- **Plugin & theme developers:** No external API changes. If you have forked or heavily customized `@wordpress/block-editor` List View internals, you will need to update prop names (`block` → `clientId`, `insertedBlock` → `insertedBlockClientId`) and replace direct block property access with `useSelect` calls.
- **Block editor users:** Smoother List View interactions in nested blocks and Navigation menus due to reduced re-render overhead and deferred `useSelect`/`useEntityBinding` calls.
- **No action required** for standard block development, theme customization, or plugin integration.

## Technical details

The diff refactors the `useListViewContext` provider and its consumers to pass `clientId` (string) instead of the full `block` object. Key changes include:

- `insertedBlock` state in `packages/block-editor/src/components/list-view/index.js` is renamed to `insertedBlockClientId`.
- `ListViewBlock` (`block.js`) and `ListViewBranch` (`branch.js`) now receive `clientId` directly, allowing `memo( ListViewBlock )` to compare a stable string reference instead of a rebuilt object.
- `block-contents.js` gates `AdditionalBlockContent` rendering by checking `insertedBlockClientId === clientId`, preventing it from mounting on non-matching rows.
- `NavigationLinkUI` (`packages/block-library/src/navigation/edit/navigation-link-ui.js`) drops the `block` prop and uses `useSelect` to fetch `insertedBlockName` and `insertedBlockAttributes` only when needed.
- `LeafMoreMenu` components in both `block-library` and `edit-site` now accept `clientId` directly and derive `blockName` via `useSelect` to check `hasBlockSupport` and `BLOCKS_THAT_CAN_BE_CONVERTED_TO_SUBMENU`.
- `BlockSettingsDropdown` (`block-settings-menu.js`) and `ListViewBlockSelectButton` also switch from `block` to `clientId`.

Before/after pattern shift:
```jsx
// Before
const [ insertedBlock, setInsertedBlock ] = useState( null );
const { block } = props;
const { clientId } = block;

// After
const [ insertedBlockClientId, setInsertedBlockClientId ] = useState( null );
const { clientId } = props;
```
Derived data that previously came from the `block` object is now explicitly fetched via `useSelect` (e.g., `getBlockName`, `getBlockAttributes`, `getBlockEditingMode`) to maintain reactivity while avoiding full object passes.

## Contribution

The change was driven by a performance optimization strategy to align the List View with React memoization best practices. Review focused on ensuring that removing the full `block` object did not break reactivity, which was resolved by explicitly fetching required properties (`blockName`, `blockVisibility`, `blockAttributes`) via `useSelect` in each consumer. The approach was accepted without major alternative debates, as passing primitive strings for identity checks is a standard pattern for stabilizing list item rendering.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

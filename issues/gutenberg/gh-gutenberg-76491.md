# #76491: Add supports for pseudo states on single block instances

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @MaggieCabrera
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`
- **Merged:** [`3822920`](https://github.com/WordPress/gutenberg/commit/382292082c267673594982838ac5624ffd20e7bc)
- **Discussion:** [#76491](https://github.com/WordPress/gutenberg/pull/76491) · 27 comments · 5 reactions

## Summary

Per-instance pseudo-state styles can now be configured on individual block instances directly in the editor, rather than only through Global Styles or `theme.json`. Blocks opt in by declaring a `"states"` array in their `block.json` supports, and the Button block is the first to adopt it with `[":hover", ":focus", ":active"]`. When a block supports states, a state-selector dropdown appears in the block card header; switching states scopes all style panels (Color, Background, Typography, Border, Dimensions) to that state. On the frontend, per-state styles are compiled into scoped inline CSS keyed to the individual block instance via a `.wp-states-{ID}` class, leaving other instances unaffected.

## Impact

**Plugin & theme developers**
- Blocks can opt in to this feature by adding `"states": [":hover", ":focus", ":active"]` (or any subset) to the `supports` object in `block.json`. No other registration change is required.
- The Button block now ships with state support enabled; editors will see the state selector in the block card header for any Button instance.
- `hasPseudoElementSupport()` in `@wordpress/block-editor` gains a second options argument `{ checkStatesSupport: true }` to distinguish state support from pseudo-element (`::before`/`::after`) support. Callers passing no options are unaffected.
- `useStylesPanel()` gains a new `currentState` parameter. Existing callers omitting it are unaffected.
- **No pseudo-class validation is performed** — an invalid value like `":foobar"` in `block.json` will silently produce broken CSS. Validation is a noted follow-up.

**Site owners / content editors**
- When editing a Button block, a three-dot dropdown in the block card header exposes Default / Hover / Focus / Active states. Styles set per-state apply only to that block instance.

**Hosting & platform**
- State styles are injected as inline `<style>` tags scoped to a `.wp-states-{ID}` class. No new database columns or option keys are introduced; styles are serialized into `attributes.style`.

**Headless & REST consumers**
- Per-state styles are stored in the block's `style` attribute under pseudo-state keys (e.g., `attributes.style[':hover']`). REST consumers retrieving raw block data will encounter these keys.

## Technical details

**`block.json` opt-in**

```jsonc
// packages/block-library/src/button/block.json
"supports": {
  "states": [":hover", ":focus", ":active"]
}
```

The `states` array is read at runtime via the new `useBlockStates(blockName)` hook, which calls `getBlockType(blockName)?.supports?.states ?? null`.

**New file: `packages/block-editor/src/components/global-styles/pseudo-elements/block-states-support.js`**

Exports three symbols now re-exported from `packages/block-editor/src/components/global-styles/pseudo-elements/index.js`:

- `useBlockStates(blockName)` — hook returning the `supports.states` array or `null`.
- `StateSelector({ isDropdown })` — reads `getBlockEditingState(clientId).pseudoState` from `blockEditorStore` and dispatches `setBlockEditingState(clientId, { pseudoState })` on selection. When `isDropdown={false}` it renders the current state label inline; when `isDropdown={true}` it renders a `<MenuGroup>` with `<MenuItem>` entries for Default and each declared state.
- `PseudoStateStylesWrapper({ children, name, clientId })` — wraps style panels; passes through `children` unchanged when no state or no states support is present, and conditionally renders a `<ShowStateOnCanvas>` toggle alongside children when a state is active.

**`BlockCard` changes (`packages/block-editor/src/components/block-card/index.js`)**

`BlockCard` now accepts a `clientId` prop. When `hasStateSupport` is true (the block has a non-empty `supports.states` and `useSupportedStyles` returns a non-empty list), the card header renders a `<DropdownMenu>` whose toggle child is a label-less `<StateSelector>` and whose menu content is `<StateSelector isDropdown={true}>`. The original `children` are moved to `<CardBody>` in this case, below the description.

**`hasPseudoElementSupport` extension (`panel.js`)**

```js
// Before
export function hasPseudoElementSupport( blockType ) { ... }

// After
export function hasPseudoElementSupport(
  blockType,
  { checkStatesSupport = false } = {}
) {
  if ( checkStatesSupport ) {
    return !! blockType?.supports?.states?.length;
  }
  // existing __experimentalPseudoElements / pseudoElements check
}
```

**`useStylesPanel` (`use-styles-panel.js`)**

Signature changes from `useStylesPanel(name, clientId)` to `useStylesPanel(name, clientId, currentState)`. Imports `useBlockStates` and includes `currentState` and `blockStates` in the `useSelect` dependency array so that switching states re-derives the value/setter functions.

**Frontend CSS output**

State styles are compiled server-side into scoped inline CSS using a `.wp-states-{ID}` class added to the block's wrapper element, so `:hover` styles target only that specific instance. The CSS compilation path (PHP side) is not shown in this diff but is described in the PR.

## Contribution

The PR was opened by @MaggieCabrera and follows on from #75627, which laid groundwork for pseudo-state handling in Global Styles. @talldan contributed additional commits. The change closes the long-standing Trac-adjacent request #38277. Reviewers @andrewserong and @ntsekouras raised three concerns: (1) the state selector's position in the block card header is visually separated from the style panels, acknowledged as a UX gap to address in a forthcoming redesign of that section; (2) no validation of pseudo-class strings in `block.json` — agreed as a follow-up given silent failure is low-risk for a developer-facing config; (3) discoverability of per-state customizations in the canvas — noted as a future enhancement. Both reviewers signed off and the PR was merged by @MaggieCabrera. The decision not to place the feature behind an experimental flag was deliberate, since the underlying capability already existed through `theme.json`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

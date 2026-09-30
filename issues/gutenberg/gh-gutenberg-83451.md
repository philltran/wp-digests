# #83451: Global Styles: Add pseudo-state controls for Navigation Link

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mikachan
- **Labels:** `[Type] Enhancement`, `Global Styles`, `[Feature] Style States`
- **Merged:** [`bb5978f`](https://github.com/WordPress/gutenberg/commit/bb5978fa6b9b46edfad124e5e1fca144072dd4c6)
- **Discussion:** [#83451](https://github.com/WordPress/gutenberg/pull/83451) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Navigation Link block was missing from the `VALID_BLOCK_STATES` list in the Global Styles UI, so the pseudo-state dropdown (Hover, Focus, Focus-visible, Active) was never rendered on the Styles → Blocks screen for that block, even though it was already listed in the other three places that track supported pseudo-states. This PR adds the missing entry, making the state controls available. The fix is a single array addition plus a unit test; no rendering or CSS-generation logic changed.

## Impact

- **Site owners / block-theme users:** The States dropdown now appears when selecting Navigation Link in Site Editor → Styles → Blocks. Selecting a state currently surfaces only the Typography panel (e.g. text-decoration), since Navigation Link supports only `typography` in this context. Colour, border, and spacing state controls are tracked separately in #79283.
- **Plugin & theme developers:** No code changes required. The block's `theme.json` and block-inspector state support were already in place; this was purely a missing UI registration.
- **No breaking changes, deprecations, or migration steps.**

## Technical details

The pseudo-state list for a block is maintained in four locations. Three already contained `core/navigation-link`:

- `VALID_BLOCK_PSEUDO_SELECTORS` in `lib/class-wp-theme-json-gutenberg.php`
- `VALID_BLOCK_PSEUDO_SELECTORS` in `packages/global-styles-engine/src/core/render.tsx`
- `VALID_BLOCK_PSEUDO_STATES` in `packages/block-editor/src/hooks/states.jsx`

The fourth, `VALID_BLOCK_STATES` in `packages/global-styles-ui/src/utils.ts`, was missing it. The diff adds:

```ts
'core/navigation-link': [
    { value: ':hover', label: __( 'Hover' ) },
    { value: ':focus', label: __( 'Focus' ) },
    { value: ':focus-visible', label: __( 'Focus-visible' ) },
    { value: ':active', label: __( 'Active' ) },
],
```

A unit test in `packages/global-styles-ui/src/test/utils.js` asserts that `getValidPseudoStates( 'core/navigation-link' )` returns the four pseudo-selector values. No changes to CSS generation, `theme.json` schema, or REST endpoints.

## Contribution

Opened by @mikachan as part of the broader Style States effort (#79283). The author flagged uncertainty about whether the omission was intentional and asked for confirmation. @talldan confirmed it was most likely a mistake rather than a deliberate exclusion, and the PR was merged with that co-author credit. The PR body notes Claude Code was used as an AI tooling aid.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

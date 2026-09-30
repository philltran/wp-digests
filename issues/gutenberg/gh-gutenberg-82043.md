# #82043: Select.Positioner: Revert to Base UI alignItemWithTrigger default

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`45ac541`](https://github.com/WordPress/gutenberg/commit/45ac541fec5d20312eb8644d7598683d9b2196ed)
- **Discussion:** [#82043](https://github.com/WordPress/gutenberg/pull/82043) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`Select.Positioner` in `@wordpress/ui` no longer hardcodes `alignItemWithTrigger={ false }`. It now defaults to `true` (Base UI's default), so the popup aligns the selected item with the trigger. When that alignment is active, the popup's `max-height` cap is removed to avoid positioning bugs that Base UI's upstream fix did not fully resolve. Consumers who want trigger-edge placement can pass `alignItemWithTrigger={ false }` explicitly.

## Impact

**Plugin & theme developers using `@wordpress/ui`**
- `Select` and `SelectControl` popups now open aligned to the selected item instead of below/above the trigger. This is a visible default behavior change; check any visual or e2e tests that assume trigger-edge placement.
- To keep the previous behavior, pass `alignItemWithTrigger={ false }` to `Select.Positioner`.
- With alignment enabled, the popup has no `max-height` cap, so very long lists are no longer height-limited by that rule.

**Site owners / end users**
- No direct action; only UI components built on the `Select` primitive are affected.

**Other item popups**
- `Combobox`, `Autocomplete`, etc. are unaffected because they don't have this feature.

## Technical details

In `packages/ui/src/form/primitives/select/positioner.tsx`, `Positioner` now destructures `alignItemWithTrigger = true` from props, instead of passing a literal `false` before the `{ ...props }` spread. The resolved value is passed to `_Select.Positioner` *after* the spread, so it always wins over any value in `props`. It also toggles a new class:

```tsx
// before
<_Select.Positioner
  { ...ITEM_POPUP_POSITIONER_PROPS }
  alignItemWithTrigger={ false }
  { ...props }
/>

// after
function SelectPositioner( { className, alignItemWithTrigger = true, ...props }, ref ) {
  <_Select.Positioner
    { ...ITEM_POPUP_POSITIONER_PROPS }
    { ...props }
    alignItemWithTrigger={ alignItemWithTrigger }
    className={ clsx( ..., alignItemWithTrigger && itemPopupStyles[ 'is-align-item-with-trigger' ], className ) }
  />
}
```

In `packages/ui/src/utils/css/item-popup.module.css`, a new rule `.is-align-item-with-trigger .popup { max-height: none; }` removes the popup height cap only when alignment is enabled. A `CHANGELOG.md` entry under Enhancements covers `Select` and `SelectControl`. No hooks, REST, or DB changes.

## Contribution

The original override existed because Base UI had positioning problems with `alignItemWithTrigger` in Gutenberg's setup, even though design (@jameskoster) had initially wanted the behavior. During review, @ciampo reproduced broken popup positioning with long lists and a deep pre-selected value (e.g. item 18 or 20 of 30). @mirka concluded the upstream fix doesn't fully handle `max-height`, so the PR was extended to drop the max-height for this case only, with the stated option to revert everything if it doesn't feel right. @jasmussen deferred to technical judgment, asking only that it be accessible. Discussion also confirmed that `defaultValue` displaying without a matching `Select.Item` is upstream Base UI behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

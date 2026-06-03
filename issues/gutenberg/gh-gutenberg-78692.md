# #78692: Tooltip migration: boot consumers + shell-level Tooltip.Provider (5/5)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Boot`
- **Merged:** [`d4fe353`](https://github.com/WordPress/gutenberg/commit/d4fe35354a3133ed912d4645ba26a3c8a27d7f44)
- **Discussion:** [#78692](https://github.com/WordPress/gutenberg/pull/78692) · 3 comments · 0 reactions

## Summary

This PR completes a five-part migration of Gutenberg's tooltip infrastructure, moving the two remaining `Tooltip` consumers in `@wordpress/boot` from the legacy `Tooltip` in `@wordpress/components` to the compositional `Tooltip` in `@wordpress/ui`. The affected components are `boot/components/site-icon-link` (codemod) and `boot/components/save-button` (manual — the codemod bails on the legacy `shortcut` prop). A shell-level `<Tooltip.Provider>` is also mounted inside `boot/components/root`, so all sibling tooltips in the boot shell now coordinate as a group (instant-open after first hover). With this merged, every `Tooltip` consumer outside `@wordpress/components` itself is on the new API, unblocking a follow-up that will mark the `@wordpress/ui` `Tooltip` as the recommended implementation.

## Impact

**Plugin & theme developers building on or extending `@wordpress/boot`**
- `@wordpress/ui` is now a **runtime dependency of `@wordpress/boot`**. The compiled module (`build/modules/boot/index.min.js`) grew **+32.3 kB (+167%)** as a result. Audit load-time budgets for boot-based apps accordingly.
- Tooltip behavior improves: after the first tooltip in a boot shell session is shown, subsequent sibling tooltips open instantly due to `<Tooltip.Provider>` coordination.
- Keyboard interaction is unchanged — the visible tooltip remains hover-only; `aria-keyshortcuts` on the save button is unaffected.

**All developers**
- No breaking API changes. No migration required on the consumer side.
- `@wordpress/components` internal `Tooltip` call sites — including `Button`'s internal tooltip and `TooltipInternalContext` — are explicitly **out of scope** and remain on the legacy implementation.

**Site owners**
- No action required.

## Technical details

Three source locations in `packages/boot` were changed:

**`boot/components/site-icon-link`** — Rewritten by the codemod from #78411. Import swapped from `@wordpress/components` to `@wordpress/ui`; import placement and a per-line `eslint-disable` directive were hand-finished.

**`boot/components/save-button`** — Migrated manually because the codemod intentionally bails on the legacy `shortcut` prop. The shortcut is now a sibling element inside `<Tooltip.Popup>` rather than a prop passed to `<Tooltip>`.

```jsx
// Before
import { Tooltip } from '@wordpress/components';
<Tooltip text={ label } shortcut={ shortcut }>
  <Button>{ children }</Button>
</Tooltip>

// After
import { Tooltip } from '@wordpress/ui';
<Tooltip>
  <Tooltip.Trigger asChild>
    <Button>{ children }</Button>
  </Tooltip.Trigger>
  <Tooltip.Popup>
    { label }
    { shortcut && (
      <span className="boot-save-button__shortcut">{ shortcut }</span>
    ) }
  </Tooltip.Popup>
</Tooltip>
```

The `.boot-save-button__shortcut` CSS mirrors the legacy `components-tooltip__shortcut` rule: `margin-left: $grid-unit-10` gated on `:not(:first-child)`, so a standalone shortcut (rendered when the post is saved and no label is beside it) stays flush left. An inline `TODO` flags a future `@wordpress/ui` `Shortcut` primitive.

**`boot/components/root`** — `<Tooltip.Provider>` is mounted directly inside `<SlotFillProvider>`, matching the placement used by the `edit-post` and `edit-site` shells (established in #78466).

```jsx
import { Tooltip } from '@wordpress/ui';
<SlotFillProvider>
  <Tooltip.Provider>
    { /* rest of boot shell */ }
  </Tooltip.Provider>
</SlotFillProvider>
```

`@wordpress/ui` is added as a runtime dependency in `packages/boot/package.json`.

## Contribution

Opened and merged by **@ciampo** as the fifth and final step in a pre-planned five-PR series (#78411 → #78466 → #78470 → #78691 → this PR). The PR was authored with Cursor assistance; the author reviewed and verified all changes. Discussion was minimal (3 comments). Two follow-ups are already tracked: codemod cleanup in #78669, and a separate PR to mark the `@wordpress/ui` `Tooltip` as the recommended implementation now that all external consumers have been migrated.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

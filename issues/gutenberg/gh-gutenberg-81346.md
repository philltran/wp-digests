# #81346: Global Styles: Keep storing shadow presets as CSS variables when custom presets exist

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Bug`, `[Package] Block editor`, `Global Styles`
- **Merged:** [`442851e`](https://github.com/WordPress/gutenberg/commit/442851efd689a0a47197c3d08c8813e9dedada2f)
- **Discussion:** [#81346](https://github.com/WordPress/gutenberg/pull/81346) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a Global Styles bug where picking a theme or default drop shadow preset stored the resolved CSS value instead of a `var:preset|shadow|<slug>` reference once any custom shadow preset existed. The border panel's slug lookup stopped at the first origin that had presets (`custom ?? theme ?? default`), so adding one custom shadow hid all theme and default presets from the search. The lookup now runs over the same merged list the popover renders, so blocks keep tracking the preset when it later changes.

## Impact

- **Site owners / editors:** Blocks that use a theme or default shadow preset after a custom shadow has been added now follow later edits to that preset in Styles → Shadows. Previously they were frozen to the value at pick time, while the popover still showed the preset as selected.
- **Plugin & theme developers:** No API change. Content already saved with resolved shadow values stays as is; only new selections are stored as preset references. Duplicate-slug presets across origins are no longer both offered in the popover (see below).
- **Themes defining overlapping slugs:** If a theme preset and a custom preset share a slug, only the most specific definition (the one with a backing CSS variable) is now offered.
- No action required.

## Technical details

Changes are in `packages/block-editor/src/components/global-styles/`:

- **`border-panel.js`:** Removes the `custom ?? theme ?? default` merge. `setShadow` now uses `useShadowPresets( settings )` and resolves the slug with `findLast` on the shadow value. The list is ordered default, theme, custom, so searching backwards picks the most specific origin when values collide. A newly added custom preset starts with the same value as the `natural` default, so this case is handled explicitly.
- **`shadow-panel-components.js`, `useShadowPresets`:** No longer prepends the synthetic `Unset` entry (`slug: 'unset'`, `shadow: 'none'`). It now also de-duplicates by slug, keeping only the last (most specific) definition, since only that one is emitted as `--wp--preset--shadow--<slug>`. The default presets are still gated by `defaultPresets`.
- **`ShadowPopoverContainer`:** Now prepends the `Unset` entry in a `useMemo` when presets exist. This keeps the display-only entry out of the lookup, so it can't be persisted as a reference to a CSS variable that is never output. `Unset` still stores the literal `none`.

Since the pre-fix behavior for a preset and its stored value differed only in the lookup list, the stored format is unchanged:

```js
// before (custom presets exist): shadow: '6px 6px 0px -3px ...'
// after:                         shadow: 'var:preset|shadow|outlined'
```

New Jest tests in `test/border-panel.js` cover theme, default, custom, and unset selection, a theme opting out of default presets, slug collisions across origins, and equal values across origins. A `CHANGELOG.md` entry is added under `BorderPanel`.

## Contribution

Authored by @jorgefilipecosta, fixing #71873, with review from @ramonjd. The description says the fix and description were drafted with AI assistance. The description flags two review points: the most-specific-origin tie-break for equal values, and dropping overridden same-slug presets from the list. The thread carries no further design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

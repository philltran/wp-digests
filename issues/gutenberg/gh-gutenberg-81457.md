# #81457: Theme: Avoid root-level relational selectors

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Performance`, `[Package] Theme`, `Backport to WP 7.1 Beta/RC`
- **Merged:** [`7a8a56b`](https://github.com/WordPress/gutenberg/commit/7a8a56b1d0c08ca1ce714b5d8a00bd1e471931f6)
- **Discussion:** [#81457](https://github.com/WordPress/gutenberg/pull/81457) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/theme` package forwarded a root `ThemeProvider`'s `cornerRadius` preset to `<html>` using relational CSS — `:root:has([data-wpds-root-provider="true"][data-wpds-corner-radius="…"])` — which showed up as repeated relational matching in style-recalculation profiles during block selection. This PR removes those `:has()` selectors: `ThemeProvider`'s existing root layout effect now mirrors `data-wpds-root-provider="true"` and `data-wpds-corner-radius="<preset>"` directly onto the owning document's `documentElement`, and the generated token CSS matches with a plain compound selector `:root[data-wpds-root-provider="true"][data-wpds-corner-radius="…"]` at the same specificity. Token resolution for root and nested providers is unchanged; the win is purely in selector-matching cost. Measured at roughly 5% improvement on block-selection style recalculation in the reporter's re-test.

## Impact

- **Plugin & theme developers (editor UI):** No action required. Corner-radius tokens (`--wpds-border-radius-xs` / `-sm` / etc.) resolve exactly as before for both root and nested `ThemeProvider`s.
- **Anyone writing CSS against the design-system attributes:** The document element itself now carries `data-wpds-root-provider="true"` and `data-wpds-corner-radius="<preset>"` when a root provider is mounted, rather than only the provider's wrapper `div`. If you wrote your own `:root:has([data-wpds-root-provider="true"]…)` rule to piggyback on the old mechanism, switch it to `:root[data-wpds-root-provider="true"]…`.
- **Iframed editor canvases:** attributes are set on the wrapper's *owning* document element, so an iframe's `<html>` gets them and the outer document does not — a regression test now asserts this.
- **Performance-sensitive platform teams:** one of several `:has()` offenders in editor style recalculation; the contributor who profiled it notes other root-level relational selectors (e.g. `body:has(.editor-editor-interface.is-distraction-free)`) remain and are being handled separately.
- **Bundle size:** `build/scripts/theme/index.min.js` +65 B, token CSS files ~-7 B each.

## Technical details

Three coordinated changes:

**1. `packages/theme/terrazzo.config.ts`** — each corner-radius mode's second selector is rewritten, which regenerates `packages/theme/prebuilt/css/design-tokens.css` (and the RTL/min variants):

```css
/* Before */
[data-wpds-corner-radius="moderate"],
:root:has([data-wpds-root-provider="true"][data-wpds-corner-radius="moderate"]) { … }

/* After */
[data-wpds-corner-radius="moderate"],
:root[data-wpds-root-provider="true"][data-wpds-corner-radius="moderate"] { … }
```

Specificity is preserved (three compound conditions either way), so cascade behaviour against the plain `[data-wpds-corner-radius]` selector is unchanged.

**2. `packages/theme/src/theme-provider.tsx`** — the `useIsomorphicLayoutEffect` that already mirrored custom properties onto `root` (the wrapper's `ownerDocument.documentElement`) now also captures `previousRootProvider` / `previousCornerRadius` via `root.getAttribute(…)`, sets both attributes, and in cleanup either restores the prior value or removes the attribute when the previous value was `null`. Because `html` is shared, the effect deliberately restores rather than wholesale-replacing attribute state. The dependency array gains `cornerRadiusPreset` (`[ cornerRadiusPreset, isRoot, themeProviderStyles ]`) so preset changes re-run the mirror.

**3. `packages/theme/src/test/theme-provider.test.tsx`** — new/extended cases cover initial forwarding of both attributes, updating on preset change (`moderate` → `pronounced`), restoring pre-existing attribute values on unmount, non-root providers leaving `document.documentElement` untouched, and the iframe case asserting the attributes land on `iframeDoc.documentElement` and not the parent document.

No public API, hook, or prop changed; `ThemeProvider`'s `isRoot` and `cornerRadius` props behave identically.

## Contribution

A follow-up to #79153 and its selector-performance discussion, implemented by @ciampo with Codex used to trace the selector path, write the change, and add the regression tests. @Mamaduka re-profiled the branch and reported ~5% improvement on block selection, explicitly walking back his earlier 17–23% figure — that measurement had also removed other relational selectors such as `body:has(.editor-editor-interface.is-distraction-free)`, which he planned to tackle in a separate PR before attempting a combined CI comparison, and he left the ship/hold call to @ciampo. @t-hamano flagged it for the 7.1 release and coordinated the backport in Slack; the automated cherry-pick to `wp/7.1` conflicted (twice), so the backport requires a hand-resolved PR against that branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #79432: UI: Disable instant overlay popup transitions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`377c12e`](https://github.com/WordPress/gutenberg/commit/377c12eafe156612ba38bc24b71c06e6fb781eff)
- **Discussion:** [#79432](https://github.com/WordPress/gutenberg/pull/79432) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/ui` package now respects Base UI's `data-instant` attribute by disabling CSS transitions when that attribute is present on a popup. Previously, `Popover.Popup` and related components always applied shared slide/fade animations, even for instant interactions like keyboard triggers or focus dismissals. This change ensures instant states skip motion entirely, aligning the WordPress UI layer with Base UI's runtime behavior.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** No code changes required. Popovers triggered via keyboard or focus will now render instantly without animation.
- **Headless/Custom UI consumers:** If you manually apply `data-instant` to Base UI popups, they will now correctly skip transitions.
- **No action required** for existing implementations; the behavior change is purely visual/performance-related and backward compatible.

## Technical details

The change adds a single CSS rule to `packages/ui/src/utils/css/dropdown-motion.module.css`:
```css
&[data-instant] {
    transition: none;
}
```
This selector targets the popup surface when Base UI attaches `data-instant` to skip motion. It overrides the shared transition properties that previously applied to all `Popover.Popup` instances. The diff does not modify JavaScript logic; it purely intercepts the CSS transition pipeline. While `Combobox` and `Autocomplete` also consume this shared class, the author confirmed `data-instant` is not currently emitted by Base UI 1.5.0 for those components.

## Contribution

The PR was authored by @ciampo and merged after review from @fcoveram. The author identified that Base UI already sets `data-instant` for instant popup states, but the shared WordPress UI motion styles were overriding it. During review, @fcoveram suggested extending the pattern to `Tooltip` with a simple fade, but that was explicitly noted as a future consideration and not included in the merge. The change shipped as a single CSS selector addition after minimal review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #77476: Common CSS: avoid false-positive border-style on custom properties

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Kgupta62
- **Labels:** `[Type] Bug`, `[Package] Block library`, `First-time Contributor`, `Global Styles`, `Backport to WP Minor Release`
- **Merged:** [`dea5de0`](https://github.com/WordPress/gutenberg/commit/dea5de0e7a29a1a8f8339969e00b06d6842c43c5)
- **Discussion:** [#77476](https://github.com/WordPress/gutenberg/pull/77476) · 6 comments · 0 reactions

## Summary

The default `border-style: solid` fallback that Gutenberg injects via `common.scss` was leaking onto elements that merely declared a CSS custom property whose name contained `border-width` or `border-color` (e.g. `--my-border-width`). This happened because the fallback selectors used substring matching (`[style*="border-color"]`), which matched any inline `style` attribute containing that string. The fix replaces each loose `*=` selector with three tighter selectors that only match when the property appears at the start of the style string or immediately after a `;` — positions where a real declaration can occur but a `--`-prefixed custom property cannot. Real `border-width`/`border-color` declarations still receive the solid-style fallback; custom properties no longer trigger a phantom border.

## Impact

- **Theme & block developers:** Fixes spurious visible borders on blocks/elements that use inline CSS custom properties named like `--my-border-width: 1px` or `--fir-list-border-color: red`. No action required — the change is purely in the generated stylesheet selectors.
- **Site owners:** Unexpected borders on custom blocks/themes disappear after the update. Legitimate inline `border-width`/`border-color` styles continue to render their default solid border style as before.
- **No breaking changes, deprecations, or migrations.** Selector specificity is unchanged because all selectors remain wrapped in `:where()`.
- **Hosting/platform note:** The fix increases the CSS payload of this fallback (three selectors per rule instead of one), shipped on every request. Reviewers flagged this as a tradeoff and noted the whole approach may eventually be replaced.

## Technical details

Change is confined to `packages/block-library/src/common.scss`. Each border-color and border-width rule (including the per-side `-top-`/`-right-`/`-bottom-`/`-left-` variants) had its single substring selector expanded into three anchored selectors inside the existing `:where()`.

Before:

```scss
html :where([style*="border-width"]) {
	border-style: solid;
}
```

After:

```scss
html
:where(
	[style^="border-width"],
	[style*=";border-width"],
	[style*="; border-width"]
) {
	border-style: solid;
}
```

The three selectors match, respectively: the property as the first declaration in the `style` attribute (`^=`), the property immediately following a semicolon (`;border-width`), and the property following a semicolon and space (`; border-width`). Since custom property names always begin with `--`, they can never appear in those positions, so `--my-border-width`/`--fir-list-border-color` no longer match. The `/*rtl:ignore*/` annotations on the right/left side rules are preserved. Keeping everything inside `:where()` means specificity stays at zero, so existing overrides behave identically.

## Contribution

Opened by first-time contributor @Kgupta62 to fix issue #70211, implementing an approach proposed by @aaronrobertshaw in PR #75546 (comment 3970212891). @talldan reviewed and approved, calling it thorough. @aaronrobertshaw raised the increased per-request CSS payload as the main concern and noted a draft alternative in #78818 that would remove the CSS fallbacks entirely in favor of edit-time and render-time fallback style injection on blocks — leaving open the possibility that this selector fix is a temporary measure. Merged in `dea5de0` and labeled for backport to a WordPress minor release.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

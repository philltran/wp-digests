# #79391: Verse block: add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Verse`
- **Merged:** [`0062d6e`](https://github.com/WordPress/gutenberg/commit/0062d6ee940a628b0b08b1e8e3480986c2855978)
- **Discussion:** [#79391](https://github.com/WordPress/gutenberg/pull/79391) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds modern `background.gradient` support to the core Verse block, replacing legacy color-based gradient handling. This resolves a CSS conflict where gradients and background images were serialized as separate declarations, causing the image override to break the intended visual layering.

## Impact

- Theme developers combining gradients and background images on Verse blocks will see corrected stacking order and properly combined CSS output.
- No migration or code changes required for existing sites; the editor automatically writes to the modern `style.background.gradient` property.
- Plugin/theme authors should note the updated `supports` declaration in `core/verse` if relying on programmatic feature detection.

## Technical details

The change modifies `packages/block-library/src/verse/block.json` to enable `"gradient": true` under both `background` and `__experimentalDefaultControls`. Previously, gradients were handled via `supports.color.gradients`, outputting a legacy `background: linear-gradient(...)` declaration that conflicted with `background-image: url(...)`. With the updated support flag, the background module emits a single combined declaration:

```css
background-image: linear-gradient(...), url(...);
```

The PR also updates block documentation (`docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/verse/README.md`) and introduces a PHP regression test verifying correct rendering of combined gradient and image backgrounds in `core/verse`.

## Contribution

Authored by @ramonjd as a direct follow-up to #75859. Reviewed and approved by @aaronrobertshaw, who confirmed the pattern would be extended to other blocks like Accordion and Pullquote in subsequent PRs. Merged on June 22 (commit `0062d6e`), with co-authorship attributed to @aaronrobertshaw in the commit metadata.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

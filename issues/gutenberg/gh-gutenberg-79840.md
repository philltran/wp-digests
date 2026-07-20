# #79840: Accordion block: add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Accordion`
- **Merged:** [`afd7bc0`](https://github.com/WordPress/gutenberg/commit/afd7bc039082b3b9eac35e9e262eb0f6c02f241d)
- **Discussion:** [#79840](https://github.com/WordPress/gutenberg/pull/79840) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Accordion block now supports background gradients through the modern `background` support path instead of the legacy `color.gradients` path. This change resolves a CSS conflict where setting both a gradient and a background image would cause the image declaration to override the gradient. By consolidating both into a single `background-image` declaration, editors can reliably layer gradients behind images without style collisions.

## Impact

- **Plugin & theme developers / block authors:** No immediate code changes required. Existing Accordion blocks with gradients will continue to render, but newly configured blocks will use the updated CSS output path.
- **Site owners & editors:** The gradient control remains hidden by default (no `__experimentalDefaultControls` added for gradients), but can be enabled via block styles or theme.json. When both a gradient and background image are set, the frontend output is now a single combined `background-image` declaration instead of conflicting `background` and `background-image` rules.
- **No action required** for existing sites; the change is backward-compatible and only affects newly configured blocks or blocks where styles are regenerated.

## Technical details

The change modifies `packages/block-library/src/accordion/block.json` to add `"gradient": true` under `supports.background`. Previously, gradients were only declared via `supports.color.gradients`, which serialized to a `background: linear-gradient(...)` declaration. When a background image was also applied, it wrote to `background-image: url(...)`, causing the image to override the gradient. With `supports.background.gradient` enabled, the block support system now writes the gradient to `style.background.gradient`, and the background support layer emits a combined declaration:

```css
background-image: linear-gradient(...), url(...);
```

The diff also updates `packages/block-library/src/accordion/README.md` and `docs/reference-guides/core-blocks/README.md` to reflect the new `gradient` support flag. No hooks, filters, REST schema changes, or database migrations are introduced.

## Contribution

Opened and merged by @ramonjd, with co-authorship from @andrewserong. The PR was submitted as a follow-up to #75859 to align the Accordion block’s gradient handling with the modern background support path. The discussion was minimal, focusing on testing the combined CSS output and verifying backward compatibility with pre-existing blocks. No alternative approaches were debated in the provided material.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

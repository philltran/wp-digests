# #79841: Pullquote block: add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Pullquote`
- **Merged:** [`5026ea3`](https://github.com/WordPress/gutenberg/commit/5026ea3c5dc696d7c97ec27807903703448770da)
- **Discussion:** [#79841](https://github.com/WordPress/gutenberg/pull/79841) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Pullquote block now routes background gradients through the unified `background` support path instead of the legacy `color.gradients` path. This resolves a CSS conflict where applying both a gradient and a background image would cause the image declaration to override the gradient. The editor now combines both into a single `background-image` declaration, ensuring both visual layers render correctly on the frontend.

## Impact

- **Block & theme developers:** No code changes required. Existing Pullquote blocks remain backward-compatible, but new gradient styling will use the updated CSS output path.
- **Site owners & editors:** The gradient control remains visible by default. Combining gradients with background images now works without visual overrides.
- **No action required** for existing sites, plugins, or themes. This is a forward-looking block support addition with no breaking changes.

## Technical details

The change modifies `packages/block-library/src/pullquote/block.json` to add `"gradient": true` under `supports.background` and `"gradient": true` under `supports.background.__experimentalDefaultControls`. Previously, gradients were only declared in `supports.color.gradients`, which serialized to a standalone `background: linear-gradient(...)` declaration. With `background.gradient` enabled, the background support system emits a combined declaration:

```css
background-image: linear-gradient(...), url(...);
```

The block documentation in `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/pullquote/README.md` is updated to list `gradient` under the `background` support group. The PR also preserves default UI visibility by adding `gradient: true` to `background.__experimentalDefaultControls`, matching the existing `color.__experimentalDefaultControls.background: true` behavior.

## Contribution

Opened and merged by @ramonjd, with co-authorship from @andrewserong. The PR was a direct follow-up to #75859, aligning the Pullquote block's gradient handling with the modern background support path. The discussion was minimal, focusing on testing the combined CSS output and verifying backward compatibility with existing blocks. No alternative approaches or design debates were documented before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

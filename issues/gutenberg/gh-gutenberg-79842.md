# #79842: Post Content block: add background gradient support

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Post Content`
- **Merged:** [`35691b3`](https://github.com/WordPress/gutenberg/commit/35691b30fc1ac13b41b27dee2dbdacd1b17276de)
- **Discussion:** [#79842](https://github.com/WordPress/gutenberg/pull/79842) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Post Content block (`core/post-content`) now supports background gradients via the modern `supports.background.gradient` API. Previously, gradients were routed through the legacy `supports.color.gradients` path, which caused CSS conflicts when combined with background images. This change consolidates gradient and image styling into a single `background-image` declaration, preventing the image from overriding the gradient.

## Impact

- **Theme & plugin developers:** No code changes or migration required. Existing templates continue to render correctly.
- **Site editors:** Can now apply both a background gradient and background image to the Post Content block without visual conflicts. The gradient control remains hidden by default (`color.__experimentalDefaultControls.background: false`) and must be manually enabled via the block settings menu.
- **No breaking changes or deprecations.** Legacy gradient styles on pre-existing blocks are preserved and will continue to work.

## Technical details

The change modifies `packages/block-library/src/post-content/block.json` to add `"gradient": true` under `supports.background`. This routes gradient styling through the modern background support system instead of the legacy color system. When both a gradient and background image are applied, the background support code now outputs a single combined declaration: `background-image: linear-gradient(...), url(...);`. Previously, the legacy path emitted `background: linear-gradient(...);` alongside `background-image: url(...);`, causing the image declaration to override the gradient. The block's `__experimentalDefaultControls` configuration remains unchanged, keeping the gradient control hidden by default. Documentation in `docs/reference-guides/core-blocks/README.md` and `packages/block-library/src/post-content/README.md` is updated to reflect the new support.

## Contribution

Opened and merged by @ramonjd, with co-authorship from @andrewserong. The PR is a direct follow-up to #75859, which initially added background image support to the block. The discussion was minimal, focusing on verifying the combined CSS output and ensuring backward compatibility with existing templates. No alternative approaches were debated; the implementation simply routes gradient support through the established background API.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

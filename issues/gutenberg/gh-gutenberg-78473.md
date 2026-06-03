# #78473: Revisions: Increase diff marker stripe contrast to 75% primary color proportion

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @himanshupathak95
- **Labels:** `[Type] Enhancement`, `[Focus] Accessibility (a11y)`, `[Package] Editor`, `[Feature] History`
- **Merged:** [`5d981ce`](https://github.com/WordPress/gutenberg/commit/5d981ce56a6a101feb885c600d0297a6bfad2570)
- **Discussion:** [#78473](https://github.com/WordPress/gutenberg/pull/78473) · 1 comments · 0 reactions

## Summary

Updates the Post Revisions minimap in the block editor to improve accessibility compliance and pattern legibility by shifting diff marker stripe ratios from 50/50 to 75/25 primary-to-accent color proportions. The stronger primary color dominance ensures the three marker colors meet the required ≥3:1 perceptual contrast against adjacent backgrounds, preventing the stripe patterns from visually blending into a flat mid-tone.

## Impact

- **Block editor users**: Revisions minimap indicators on the right edge will render with noticeably stronger primary color presence, making added, removed, and modified block states easier to distinguish quickly.
- **Plugin & theme developers**: No action required. This change is isolated to the `@wordpress/editor` package's frontend styling and does not affect PHP APIs, REST endpoints, block registration schemas, or published frontend markup.
- **Hosting & platform teams**: No configuration, migration, or cache-busting steps are required.

## Technical details

Modifies CSS rendering rules in `packages/editor/src/components/post-revisions-preview/style.scss` for the revisions minimap marker classes:

- `.is-removed`: Shifts from a 6px repeat period (`3px` primary / `3px` accent) to an 8px period (`6px` primary / `2px` accent). The repeating gradient now reads: `repeating-linear-gradient(45deg, #d63638, #d63638 6px, rgba(#d63638, 0.45) 6px, rgba(#d63638, 0.45) 8px)`.
- `.is-modified`: Applies the identical 75/25 proportional shift to its `-45deg` counter-diagonal gradient and updates the base hex from `#dba617` to `#9a7000`.
- `.is-added`: Replaces the solid background color with `#008a20` (previously `#00a32a`).

All modifications are scoped to CSS class definitions. No JavaScript hooks, PHP filters, or database schemas are altered.

## Contribution

Opened by @himanshupathak95 as a direct follow-up to PR #77904 and part of epic #77530, the PR addresses review feedback regarding perceptual contrast in the revisions minimap. Co-authored by @im3dabasia, the diff was finalized to implement a strict 75/25 primary-to-accent stripe ratio. Merged as `5d981ce` after passing automated checks and minimal review discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

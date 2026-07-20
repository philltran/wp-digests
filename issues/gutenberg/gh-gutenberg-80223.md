# #80223: Add contrast checking for viewport and pseudo states

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`df3d1f8`](https://github.com/WordPress/gutenberg/commit/df3d1f8dca80cbc628a1274f56bd9f5c8aeb5336)
- **Discussion:** [#80223](https://github.com/WordPress/gutenberg/pull/80223) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The block editor’s contrast checker now evaluates color combinations for responsive viewport states (mobile, tablet) and pseudo-states (hover, focus). Previously, the checker was disabled when editing these states, allowing authors to set inaccessible color combinations without warning. This enhancement ensures accessibility validation applies consistently across all style states.

## Impact

- **Block editor users & theme/plugin developers:** Contrast warnings will now appear in the Typography and Background panels when editing responsive or pseudo-state colors.
- **No breaking changes or API deprecations.**
- **No migration or code changes required.

## Technical details

The change removes the `! isStateSelected` guard from the `enableContrastChecking` boolean in three block editor hooks: `packages/block-editor/src/hooks/background.js`, `packages/block-editor/src/hooks/elements.js`, and `packages/block-editor/src/hooks/typography.js`. In `background.js`, the diff also introduces a `value` variable that resolves to `getStyleForState( style, selectedState )` when `isStateSelected` is true, ensuring the contrast logic evaluates the correct state-specific color values rather than the base `styleValue`. An e2e test was added to `test/e2e/specs/editor/various/contrast-checker.spec.js` to verify warnings appear in both Typography and Background panels for viewport states.

## Contribution

Opened by @tellthemachines to address issue #78305 and merged after review by @ramonjd, @andrewserong, and others. The PR description notes it was primarily a matter of toggling flags that disabled contrast checking for style states. Reviewers confirmed the fix works for link colors and viewport states, with minor testing notes around button elements and UI bouncing noted as out of scope. Co-authored by @andrewserong, @ramonjd, @talldan, and @juanfra.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

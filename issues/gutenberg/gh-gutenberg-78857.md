# #78857: Revert navigation morph & playlist commits pushed directly to trunk

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Task`, `[Package] Block library`
- **Merged:** [`a7aa23f`](https://github.com/WordPress/gutenberg/commit/a7aa23f024fb711061841230f65072f58ee43a6f)
- **Discussion:** [#78857](https://github.com/WordPress/gutenberg/pull/78857) · 6 comments · 0 reactions

## Summary

This pull request reverts a batch of 11 commits that were accidentally pushed directly to the Gutenberg `trunk` branch, which caused continuous integration pipelines to fail. The rolled-back changes introduced experimental navigation overlay morph animations, playlist block waveform visualization styles, and associated JavaScript/CSS fixes. Reverting restores the pre-merge baseline for the block library to stabilize the build pipeline while proper review workflows are reinstated.

## Impact

- **Theme & plugin developers:** No breaking API changes or PHP/REST modifications. The rollback removes newly added front-end assets related to navigation morphs and playlist visual variations.
- **Site builders / end users:** Navigation overlays will no longer animate between hamburger and close icons, and playlist blocks will revert to their previous default styling without waveform visualization.
- **Platform & CI:** Restores a stable `trunk` build state, eliminating the pipeline failures triggered by the unintended direct push.
- **Action required:** None. This is a front-end asset rollback with no database changes, schema modifications, or hook alterations.

## Technical details

The diff consists of `git revert` operations targeting `packages/block-library/src/navigation/` and `packages/block-library/src/playlist/`. It strips WAAPI-driven navigation morph logic, removes playlist waveform CSS variations, and restores the original opacity handling for the morph close button. The `withScope` wrapper previously added around navigation morph `onComplete` callbacks is also removed. Build artifacts reflect this reduction: `build/modules/block-library/navigation/view.min.js` shrinks by ~1.06 kB, and associated CSS files in `build/styles/block-library/navigation/` are reduced across minified and unminified builds. No PHP files, block.json fields, or REST routes are modified; the change is strictly a rollback of client-side block library assets.

## Contribution

Opened and merged by @t-hamano to address CI failures resulting from 11 commits being pushed directly to `trunk` without following standard review gates. The author generated the revert via `git revert`, verified the build size reductions, and merged at commit `a7aa23f0`. No alternative strategies or extended design debates were documented; the objective was strictly a rapid rollback to stabilize the branch.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

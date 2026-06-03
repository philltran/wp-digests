# #78489: Media Editor Experiments: Tidy up by removing old pathways to the media editor experiment

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Feature] Media`, `[Package] Editor`, `[Type] Experimental`, `[Package] Edit Site`
- **Merged:** [`3f92688`](https://github.com/WordPress/gutenberg/commit/3f92688396c4122d6b0a50e563f3b20d2a004e6b)
- **Discussion:** [#78489](https://github.com/WordPress/gutenberg/pull/78489) · 4 comments · 1 reactions

## Summary

This pull request removes legacy experimental pathways for media editing within the `editor` and `edit-site` packages. As part of a broader consolidation (#73771), media editing state that previously mutated the main block editor's internal state has been unwound in favor of dedicated modal and route-based workflows. The cleanup eliminates unused experiment triggers without affecting the newly established media editor modal or its subsequent implementation.

## Impact

- Plugin & theme developers: No breaking changes to public APIs or published components. Any custom logic relying on the legacy, experimental "media editing" state toggle inside the block editor will no longer trigger that pathway.
- Editor/Platform teams: Removes outdated state management logic tied to early media experiments, reducing cognitive overhead for core maintainers.
- Action required: None for typical consumers of the editor packages. Developers explicitly handling custom editor states related to legacy media experiments should verify their setups against the new modal/dedicated route flow.

## Technical details

- Targets `packages/editor` and `packages/edit-site` to strip no-longer used experiment pathways.
- Replaces legacy in-editor media editing mutations with a cleaner separation. The author explicitly notes that mutating the block editor's state for media edits was harder to manage handoffs, whereas modal-driven workflows provide a clearer entry/exit point.
- The diff unwinds conditional checks and state toggles tied to the old media editor experiment, aligning package code with the consolidation established in PR #77994. No new hooks or schema changes are introduced; this is a structural cleanup of internal experiment tracking.

## Contribution

Opened and merged by @andrewserong as part of the Media Editor Experiments initiative. Co-authored with @ramonjd on merge commit `3f92688`. The author explicitly rejected maintaining the in-editor state mutation approach, stating that it made handoffs between block and media editing states messy. The final implementation prioritizes modal-driven editing (triggered via the Crop button) and dedicated routes, leaving only those pathways active.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

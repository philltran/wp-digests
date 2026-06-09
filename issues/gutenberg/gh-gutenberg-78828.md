# #78828: env: Replace extract-zip with adm-zip to fix hang on Node 24.16

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @torounit
- **Labels:** `[Type] Bug`, `[Tool] Env`
- **Merged:** [`727142a`](https://github.com/WordPress/gutenberg/commit/727142a0661bcf00df5f674d3340ab3b28a95037)
- **Discussion:** [#78828](https://github.com/WordPress/gutenberg/pull/78828) · 6 comments · 0 reactions

## Summary

WordPress's `@wordpress/env` package resolves a silent environment startup hang on Node.js 24.16+ by replacing `extract-zip` with `adm-zip`. The update fixes a stream backpressure stall that caused `wp-env start` to exit cleanly with code 0 while leaving partial `.zip` and `.temp` residues when fetching URL-based core or plugin archives. This ensures reliable local environment provisioning without requiring manual intervention or configuration tweaks.

## Impact

- **CLI Users & Local Dev Environments**: `wp-env start` will no longer silently abort on Node 24.16+ when pulling remote zip sources. Existing `.wp-env.json` configurations require zero changes.
- **Package Maintainers & Dependency Managers**: The `extract-zip`, `yauzl`, and `fd-slicer` packages are removed from the `@wordpress/env` dependency tree. Regenerate `node_modules` after updating to prevent stale stream dependencies from conflicting with the new pure-JS extractor.
- **No action required** for end users, plugin/theme developers, or REST API consumers; this is strictly a build/tooling fix for local CLI environments.

## Technical details

The diff swaps `extract-zip` for `adm-zip` within the `@wordpress/env` package to bypass a backpressure stall in `fd-slicer@1.1.0` (piped into `zlib.InflateRaw`) on libuv 1.52.1 (Node 24.16). The extraction logic is updated to call `adm-zip`'s `extractAllToAsync()` method, preserving the non-blocking async contract that previously caused the `await extractZip(...)` promise to hang indefinitely. The change removes ~26 lines from `package-lock.json` (eliminating `extract-zip`, `yauzl`, and their transitive sub-deps) while adding `adm-zip`, which was already a leaf dependency of `@wordpress/scripts`. Small zips previously bypassed the stall because they resolved before backpressure triggered; larger archives now complete cleanly without leaving partial file residues.

## Contribution

Opened and merged by @torounit with co-authors t-hamano, mukeshpanchal27, and westonruter. The PR closes #78762 directly, noting that `adm-zip` was already present in the project's dependency graph via `@wordpress/scripts` and carried zero transitive dependencies. Investigation confirmed the root cause was a libuv backpressure regression rather than a core Node.js API change. The fix was reviewed with AI assistance but manually verified before merging at commit `727142a`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

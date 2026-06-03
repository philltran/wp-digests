# #78560: Add cherry-pick script and update release tools in package.json

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @kushagra-goyal-14
- **Labels:** `[Type] Build Tooling`
- **Merged:** [`9908dd1`](https://github.com/WordPress/gutenberg/commit/9908dd1ec8fe8bc0e8a089d98d00be16e6d30e62)
- **Discussion:** [#78560](https://github.com/WordPress/gutenberg/pull/78560) · 6 comments · 0 reactions

## Summary

The Gutenberg monorepo has consolidated internal build tooling by migrating the `cherry-pick` CLI script from the root `bin/` directory into the dedicated `@wordpress/release-tools` workspace. This change centralizes release utilities and removes two unnecessary root-level dependencies (`node-fetch` and `@octokit/rest`) that are now consumed internally by the tools package. The migration ensures consistent script discovery and aligns with the broader repository build architecture refactoring.

## Impact

- **Gutenberg maintainers & CLI users**: The original path `bin/cherry-pick.mjs` is removed; scripts must now invoke the entry via the `@wordpress/release-tools` workspace package.
- **Release automation workflows**: Root-level execution of `npm run other:cherry-pick` has been refactored to delegate to the release tools workspace, preserving the same CLI behavior but changing the underlying file resolution.
- **Theme/plugin developers & site owners**: No action required. This is strictly internal repository tooling and does not affect the WordPress core runtime, public APIs, or front-end/editor surfaces.
- **Dependency management**: Root `devDependencies` are now lighter; `node-fetch` and `@octokit/rest` are no longer declared at the monorepo root and should be removed from local `package.json` files if manually added.

## Technical details

The unified diff performs three linked operations:
1. **File relocation**: `bin/cherry-pick.mjs` is renamed/moved to `tools/release/cherry-pick.mjs` (100% similarity, no code alterations).
2. **Root package.json updates**: The `other:cherry-pick` script entry is updated from `"node ./bin/cherry-pick.mjs"` to `"npm run --workspace @wordpress/release-tools cherry-pick --"`. Concurrently, `@octokit/rest` (v16.26.0) and `node-fetch` (v2.7.0) are removed from the root `devDependencies` block.
3. **Build config cleanup**: `cherry-pick.mjs` is stripped from the include array in `bin/tsconfig.json`. The new entry point is explicitly added to `tools/release/package.json` under its `scripts` object as `"cherry-pick": "node ./cherry-pick.mjs"`.
No public-facing hooks, REST routes, or block schemas were altered.

## Contribution

Opened by @kushagra-goyal-14 as a follow-up to organization efforts in #75041, the PR migrates the cherry-pick script into the newly introduced `@wordpress/release-tools` workspace. During review, @manzoorwanijk proposed and approved stripping `node-fetch` and `@octokit/rest` from the root `package.json`, as those dependencies are now scoped to the release tools package. Merged under commit `9908dd1`, the change only refactors paths and script delegation without altering the CLI's underlying logic.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

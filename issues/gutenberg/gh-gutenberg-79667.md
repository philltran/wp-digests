# #79667: Revert npm v11 supply-chain settings (allow-git, allow-remote, allowScripts)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @manzoorwanijk
- **Labels:** `[Type] Security`
- **Merged:** [`67245e2`](https://github.com/WordPress/gutenberg/commit/67245e20185d53a7f080e6d5f2513c4848232cd9)
- **Discussion:** [#79667](https://github.com/WordPress/gutenberg/pull/79667) · 3 comments · 0 reactions
- **Usefulness:** 2/5

## Summary

Reverts npm v11 supply-chain configuration added in PR #79614, removing `allow-git` and `allow-remote` directives from `.npmrc` and the `allowScripts` object from `package.json`. The change resolves CI failures on Node 24, whose bundled npm version predates support for these package installation controls.

## Impact

- **Gutenberg build/CI maintainers:** Restores compatibility with Node 24’s bundled npm in the repository pipeline. Local development environments that adopted the v11 settings should remove them to match the main branch.
- **Plugin & theme developers / End users:** No impact. These are internal build tooling configurations that do not affect published packages, the WordPress runtime, or developer-facing APIs.

## Technical details

The diff explicitly strips four lines from `.npmrc` (`allow-git = none` and `allow-remote = none`) and deletes the entire `"allowScripts"` block from `package.json`, which previously scoped script execution flags for packages like `@parcel/watcher`, `esbuild`, `fsevents`, and `nx`. Removing these keys reverts npm’s package resolution and lifecycle behavior to defaults. The change intentionally leaves the `package-lock.json` structure untouched, preserving the independent cleanup from PR #79614.

## Contribution

Opened by @manzoorwanijk to address Node 24 CI failures, with review/co-merged contributions from @tyxla. The author explicitly noted that the accompanying `package-lock.json` cleanup remained in place as an independent change. After confirming CI stability, the revert was merged without additional feature additions.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

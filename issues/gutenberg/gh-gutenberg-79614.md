# #79614: Opt in to npm v11 supply-chain security features

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @manzoorwanijk
- **Labels:** `[Type] Security`
- **Merged:** [`1b28ba0`](https://github.com/WordPress/gutenberg/commit/1b28ba0a1ded0c86502d3a403ef96ed066a251aa)
- **Discussion:** [#79614](https://github.com/WordPress/gutenberg/pull/79614) · 7 comments · 0 reactions
- **Usefulness:** 2/5

## Summary

Pull request #79614 introduced npm v11 supply-chain security hardening for the Gutenberg build environment by disabling git/remote dependencies and blocking package install scripts by default. The change was merged but immediately reverted in PR #79667 after triggering test suite failures on Node 24, so it did not persist in canonical builds.

## Impact

- **Gutenberg core & plugin/theme developers:** No lasting impact. The security policies for `allowScripts` and remote/git dependencies were rolled back due to Node 24 CI incompatibility.
- **Hosting & platform teams:** None. This is a local development/build toolchain adjustment.
- **Action required:** None. The change was reverted and does not ship in current repository state.

## Technical details

The diff modifies three build configuration files to enforce stricter dependency resolution:
- `.npmrc` gains `allow-git = none` and `allow-remote = none` to block dependencies resolved via git URLs or arbitrary remotes.
- `package.json` adds an `allowScripts` policy that defaults to `false`, explicitly whitelisting only `leveldown` (due to missing Apple Silicon prebuilds) and `esbuild` (to work around npm CLI symlink drops). All other scripts (`@parcel/watcher`, `core-js`, `nx`, etc.) are denied.
- `package-lock.json` removes an extraneous `esbuild@0.25.12` entry nested under a stale Storybook dependency.
The author deliberately omitted `strict-allow-scripts` until CI is upgraded to npm v11, leaving the policy advisory-only for now. Because npm v10 ignores these settings, the changes function as a no-op in existing CI environments.

## Contribution

Opened by @manzoorwanijk as a follow-up to #78191, the PR was reviewed and merged (commit `1b28ba0`) with co-authors Mamaduka, tyxla, and aduth. Initial testing on npm v11.18 passed locally, but CI failures emerged on Node 24 environments lacking the latest npm CLI. The author noted the dependency mismatch between Node runtime versions and npm feature availability, then opened PR #79667 to revert the commit.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

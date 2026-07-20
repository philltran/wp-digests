# #80187: Release: Make npm publishing rerunnable

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `Gutenberg Plugin`
- **Merged:** [`8eec0b0`](https://github.com/WordPress/gutenberg/commit/8eec0b027012559a9258c80794ab6de07b138567)
- **Discussion:** [#80187](https://github.com/WordPress/gutenberg/pull/80187) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg release tooling now validates existing npm registry entries against the prepared release commit and expected dist-tag before attempting to publish. This prevents accidental overwrites or resume failures when a previous publish run partially succeeded and npm stripped SemVer build metadata. The change ensures the `lerna publish` step is safely rerunnable without corrupting release state.

## Impact

- **Gutenberg plugin maintainers & release engineers:** The internal release workflow now enforces strict registry validation. No action required for end users or external plugin/theme developers.
- **No breaking changes for external consumers.** The modified functions and constants are internal to `tools/release/commands/packages.js` and only affect the Gutenberg plugin release process.

## Technical details

In `tools/release/commands/packages.js`, `runNpmPublishPreflight` now accepts `distTag` and `publishCommit` parameters. It runs `npm view <pkg>@<ver> version gitHead dist-tags --json` and validates that the registry `version`, `gitHead`, and `dist-tags[distTag]` match expectations. If all match, it returns the package name to skip publishing. `publishVersionedPackagesToNpm` retrieves the current commit via `git.revparse( [ 'HEAD' ] )`, passes it to `lerna publish from-package --git-head <sha>`, and runs a post-publish verification phase before pushing Git metadata.

The `--build-metadata` flag was removed from the `lerna version` command for `next` releases to prevent local/registry version mismatches:

```diff
- npx lerna version pre${ minimumVersionBump } --preid next.v.${ timestamp } --build-metadata ${ beforeCommitHash } --no-private --no-push ${ yesFlag }
+ npx lerna version pre${ minimumVersionBump } --preid next.v.${ timestamp } --no-private --no-push ${ yesFlag }
```

`runNpmReleaseGitPushPhase` and `NPM_RELEASE_GIT_PUSH_ATTEMPTS` were renamed to `runNpmReleasePhase` and `NPM_RELEASE_PHASE_ATTEMPTS`. Documentation in `docs/contributors/code/release/package-release-and-core-updates.md` was updated to reflect the new manual recovery commands.

## Contribution

Opened and merged by @ciampo (with co-authors @ramonjd and @tyxla). Extracted from #79906 and builds on #79905. The PR focuses on making the npm publish step idempotent by validating registry state against the prepared commit. It passed CI checks and was merged without notable design debates in the provided discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

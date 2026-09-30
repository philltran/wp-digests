# #81447: Env: Update git to latest commit when `--update` passed

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Bug`, `[Tool] Env`
- **Merged:** [`9045ead`](https://github.com/WordPress/gutenberg/commit/9045eadf0b385a4ae4effc758479b45d0468658c)
- **Discussion:** [#81447](https://github.com/WordPress/gutenberg/pull/81447) · 2 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/env` now hard-resets a git source's local clone to the fetched commit when `--update` is passed. Previously, a source pointing at a branch (e.g. `WordPress/WordPress`) stayed at whatever commit it was first cloned at, because `git checkout` on an existing local branch does not fast-forward it. This is a fix for a regression introduced in #28848.

## Impact

- **Plugin/theme/core contributors using `wp-env`:** `wp-env start --update` (and scripts wrapping it, such as Gutenberg's `test:unit:php:setup -- --update`) now actually pulls git-based sources up to date. Local test runs will more faithfully reproduce failures caused by upstream WordPress Core changes.
- **Side effect to expect:** Tests that previously passed locally only because the clone was stale may now fail. That is the intended outcome.
- **Local modifications:** The reset is `--hard`, so any uncommitted changes or local commits on a wp-env-managed clone under `~/.wp-env` will be discarded on update. These clones are ephemeral and not intended for hand edits.
- **No config changes required.** Users who worked around the bug by manually deleting or updating clones in `~/.wp-env` no longer need to.

## Technical details

The change is in `packages/env/lib/download-sources.js`, in `downloadGitSource()`. After the existing `git.checkout( source.ref )` call, the diff adds:

```js
log( 'Resetting to the fetched commit.' );
await git.reset( [ '--hard', 'FETCH_HEAD' ] );
```

The function already fetches the ref before checkout. If the clone already exists, checking out an existing local branch leaves it at its old commit, so the fetched commit was never applied. `FETCH_HEAD` is used rather than `source.ref` because `source.ref` is `undefined` when the source string has no `#ref` part, in which case the repository's default branch is the target. This also works when `ref` is an explicit tag.

Per the PR description, before #28848 the code did a forced checkout of `FETCH_HEAD`, which left the repo in detached HEAD state. The new approach keeps the branch checked out and moves it, so no detached HEAD. A `CHANGELOG.md` entry was added under Bug Fixes. There are no API, hook, or config changes, and no tests are included in the diff.

## Contribution

Authored by @aduth, who traced confusing trunk test failures (reported in WordPress Slack threads) to the stale local clone. The PR was written with Claude Code and reviewed manually. It was designed to be tested alongside #81444, whose PHP 8.5 failure it makes reproducible locally. @Mamaduka is credited in the props list. The record shows no design debate beyond the choice of `git reset --hard FETCH_HEAD` over restoring the earlier forced checkout.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

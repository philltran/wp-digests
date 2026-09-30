# #82383: Cherry-pick PRs for WP 7.1.1

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @draganescu
- **Labels:** `[Type] Task`, `[Package] Editor`, `[Package] Block library`, `[Package] Block editor`, `[Package] Edit Post`, `[Tool] Env`, `[Package] E2E Tests`, `[Package] Project management automation`, `[Package] Media Utils`, `[Package] wp-build`
- **Merged:** [`dd294ab`](https://github.com/WordPress/gutenberg/commit/dd294ab86ceaf9f3d7d42a09b78f399238ce8826)
- **Discussion:** [#82383](https://github.com/WordPress/gutenberg/pull/82383) · 17 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

This pull request backports a batch of Gutenberg fixes labelled "Backport to WP Minor Release" onto the `wp/7.1` branch so they can ship in the Gutenberg packages bundled with WordPress 7.1.1. It covers roughly 19 cherry-picked PRs across the editor, block library, block editor, edit-post, media-utils and upload-media packages, plus wp-env and CI fixes needed to keep the branch green. Three of the included PRs also carry PHP changes that had to be committed separately to wordpress-develop's `7.1` branch.

## Impact

**Site owners**
- Receive the included fixes when they update to WordPress 7.1.1. No action is required beyond updating.

**Plugin & theme developers**
- No new public APIs, deprecations, or removals are listed in this PR. Behavior changes are limited to bug fixes in the bundled editor packages.
- The block-library bundle grew slightly. The size report shows `accordion-panel/style*.css` going from about 118 B to about 160 B, so if you override Accordion Panel styles, spot-check them after updating.
- Media handling is touched: #81947 and #82265 change upload and finalize behavior, including `HeicUnsupportedError` handling in `upload-media` and a re-read of the post in `finalize_item()`. Plugins that extend the attachments REST controller or client-side media processing should test against 7.1.1 RC1.

**Hosting & platform / core contributors**
- Three PHP changes live in wordpress-develop, not in the package build: #81908 (changeset 63474), #82335 (changeset 63523) and #81947 (changeset 63572). The package update is staged as WordPress/wordpress-develop#13422 (draft), to be re-pinned to the `wp/7.1` tip after this PR is rebase-merged.

**Testers**
- 7.1.1 RC1 is the point to verify the fixes and smoke-test the editor.

## Technical details

The PR is a stack of cherry-picked commits and must be merged with **Rebase and merge**, not squash, to preserve per-commit history.

Adaptations the description notes because `wp/7.1` diverges from trunk:
- **#82011**: one-line test change (fac03fe752a), because the empty canvas is `role=button` on this branch. #81231 is not on it.
- **#81947** (e2a9c8d8eef): trunk's Vitest tests and mocks were converted to Jest, since `wp/7.1` still runs Jest. The `finalize_item()` post re-read was inserted by hand into the branch's attachments controller, which lacks the sub-size provenance code found on trunk. The PHP tests apply unchanged.
- **#82265** (3f82d426679): the same Jest conversion (`vi.mock`/`vi.fn` back to `jest.mock`/`jest.fn`), with `HeicUnsupportedError` kept real via `jest.requireActual` so the `instanceof` branch is exercised.
- **#82465**: applied to the `.js` files, because trunk renamed them to `.jsx` in #80990, which is not on `wp/7.1`.
- **#82478**: wp-env Docker build fix needed for the PHP CI jobs.
- **#82249**: CI-only, taken from trunk. The e2e "Report to GitHub" job checks the `report-flaky-tests` action out from trunk, whose interface #82249 changed, so every e2e run on the branch failed. The branch has neither the sharded performance workflow nor `.github/setup-npm`, and pins Playwright 1.61 and Jest 29, so the workflow changes were ported onto its job layout, with follow-ups 5ea23e4d5f0 and 018ee8454d0.

The visible diff is mostly CI workflow changes from #82249. Bundle-size, e2e, label-enforcement and first-time-contributor automation now write to a single "PR meta" comment through a shared `tools/pr-meta` action:
- Each writer job uses `concurrency: group: pr-meta-<PR number>` with `cancel-in-progress: false` and `queue: max`, and `.github/actionlint.yml` ignores the `unexpected key "queue"` lint error.
- `bundle-size.yml` now sets `use-check` (non-fork PRs only), uploads the report as a `pr-meta--bundle-size` artifact, and updates the `bundle-size` section in a separate `pr-meta` job.
- `end2end-test.yml` replaces the `report-to-issues` job with a flaky-tests section rendered by `./packages/report-flaky-tests` and posted via `pr-meta`, on pull requests only.
- `enforce-pr-labels.yml` drops `mheap/github-action-required-labels` for a `gh api` check that requires exactly one `[Type]` label, adds `opened` and `reopened` triggers, and fails the job if the count is not 1.
- `pull-request-automation.yml` gains `welcome` and `account-link` jobs.
- `workflow-lint.yml` also watches `tools/**/action.yml`.

Backport changelog entries such as `backport-changelog/7.1/13398.md` track the PHP changes for wordpress-develop. The diff is truncated, so the source changes for the other cherry-picked PRs are not visible here.

## Contribution

@draganescu opened the PR as the 7.1.1 cherry-pick vehicle (core ticket 66037), ahead of the RC1 date. @adamsilverstein, @t-hamano and @ramonjd added further cherry-picks during the review window. Notably, #82478 was added to fix a CI error and #82249 to unblock e2e runs, and #82403 was left unmerged and not cherry-picked. Claude was used for parts of the cherry-picking (#81947, #82265), and the description records its adaptation notes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

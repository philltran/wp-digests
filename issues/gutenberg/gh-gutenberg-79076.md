# #79076: Dependabot: Add npm entry so security update PRs can be rebased

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @manzoorwanijk
- **Labels:** `[Type] Security`
- **Merged:** [`29f4f14`](https://github.com/WordPress/gutenberg/commit/29f4f14a32d1e61969a6db0c46bb2e0ed8a9d3d7)
- **Discussion:** [#79076](https://github.com/WordPress/gutenberg/pull/79076) · 7 comments · 0 reactions
- **Usefulness:** 2/5

## Summary

The Gutenberg monorepo now includes an `npm` ecosystem entry in `.github/dependabot.yml` to properly own Dependabot security update PRs. Previously, security alerts for npm packages triggered PRs that failed to rebase or recreate because they lacked a matching configuration block. This change gives those PRs a config owner, enabling bot commands like `@dependabot rebase` while explicitly keeping automated version updates disabled to prevent repository noise.

## Impact

- **Repository maintainers & CI/CD operators**: Dependabot security PRs for npm packages will now correctly accept `@dependabot rebase` and `@dependabot recreate` comments, and will automatically receive the `dependencies` and `[Type] Build Tooling` labels.
- **Plugin/theme developers & end users**: No direct impact. This is an internal repository configuration change.
- **Action required**: None for external consumers. Maintainers should monitor the next npm security PR to verify label assignment and rebase functionality.

## Technical details

The diff modifies `.github/dependabot.yml` by appending a new `updates` block for `package-ecosystem: 'npm'` rooted at `directory: '/'`. It sets `schedule.interval` to `'daily'`, `open-pull-requests-limit` to `0` (which suppresses routine version bumps but leaves security patches unaffected), and attaches `labels: ['dependencies', '[Type] Build Tooling']`. This mirrors the existing `github-actions` configuration structure. The change does not alter any PHP, JavaScript, or block editor APIs; it solely affects GitHub Actions workflow behavior and Dependabot's YAML parser expectations.

## Contribution

Opened to resolve a recurring workflow blocker where Dependabot security PRs for npm packages could not be rebased due to a missing config owner. During review, a maintainer questioned whether enabling routine version updates alongside security patches would better serve the monorepo, but the author deferred that evaluation to post-merge. Another maintainer clarified that the configuration change would only apply to newly generated PRs, not retroactively fix stale ones, leading to the decision to merge with version updates disabled. The approach was accepted after confirming it would stabilize the rebase workflow for future security alerts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

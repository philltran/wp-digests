# Recap: Restoring removed version history.

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Dennis Snell
- **Published:** 2026-06-17
- **Tags:** `General`, `build-test-tools`, `git`, `post-mortem`, `svn`
- **Link:** [https://make.wordpress.org/core/2026/06/17/recap-restoring-removed-version-history/](https://make.wordpress.org/core/2026/06/17/recap-restoring-removed-version-history/)
- **Usefulness:** 3/5

## Summary

A significant repository maintenance effort resolved infrastructure breakage caused by build script changes that moved specific PHP files out of SVN version control and into `.gitignore`. The fix merges a dedicated restore branch into `trunk` to reintroduce these PHP files back into the SVN repository, replaying their full upstream Gutenberg sync history as sequential commits.

## Impact

- **Core contributors & DevOps:** Restores broken Git-to-SVN synchronization and rebuilds CI/CD pipelines that depend on checking out commits and executing build steps.
- **Developers using `wordpress-develop`:** Recovers lost version history for affected files, enabling IDEs, GitHub's UI, and standard `git log` workflows to correctly track changes and debug regressions introduced since January.
- **Hosting & Platform teams:** Stabilizes the `server2` build process output; no immediate code changes required beyond pulling the updated repository state.

## Technical details

The resolution addresses side effects from commit `[61438]`, which had removed critical PHP files from version control in favor of a Gutenberg-sync build step (`npm run build:dev`). The fix merges a branch into `trunk` that replays the intervening history of changes, effectively reverting the `.gitignore` and `svn:ignore` directives for these specific files. This reconnects the dual-source-of-truth gap between Core and the Gutenberg plugin by ensuring the committed copies exist in SVN again. The merge replays sync commits sequentially into SVN, allowing tools to trace defects back to specific Gutenberg pull-ups. Future maintenance notes highlight unresolved discrepancies: harmonizing conflicting git/SVN ignore lists (#64971) and removing stale files from `wp-admin/includes/update-core.php`'s `$_old_files` array (#65418).

## Contribution

Following initial build script changes committed on January 5, integration failures and severed version history were immediately reported. A restoration strategy was proposed in late February and simulated extensively via custom scripts to prototype the Git branch and mirror it to a test SVN repository (`develop.svn.wordpress.org`). After numerous iterations and manual coordination during deployment on March 26, the restore branch was successfully merged into `trunk`, replaying sync commits back into SVN without triggering unexpected bot-driven build outputs. Key contributors include @desrosj, @aidvu, @abbe, @dmsnell, and @swissspidy.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

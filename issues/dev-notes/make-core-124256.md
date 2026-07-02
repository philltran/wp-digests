# Guidelines for Syncing Code From Gutenberg Into WordPress Develop

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jonathan Desrosiers
- **Published:** 2026-06-30
- **Tags:** `General`
- **Link:** [https://make.wordpress.org/core/2026/06/30/guidelines-for-syncing-code-from-gutenberg-into-wordpress-develop/](https://make.wordpress.org/core/2026/06/30/guidelines-for-syncing-code-from-gutenberg-into-wordpress-develop/)
- **Usefulness:** 2/5

## Summary

This post establishes the official workflow for syncing Gutenberg plugin code into the core `wordpress-develop` repository, shifting from npm package imports to downloading built assets via the `build-plugin-zip.yml` GitHub Actions workflow. It defines distinct alpha and beta/RC sync cadences, mandates pinning `package.json`'s `gutenberg.sha` to specific `wp/X.Y` branch SHAs during release stabilization, and standardizes Trac ticketing scopes and commit message templates for hash bumps.

## Impact

['- Core committers & release engineers must adopt the new alpha cadence (weekly sync one week post-Gutenberg release) and strictly pin trunk to `wp/X.Y` branch hashes during beta/RC phases to prevent unready feature work from leaking into numbered SVN branches.', '- Plugin/theme developers & platform teams will experience more predictable dev build stability during alpha periods, but no API-level changes or code migration steps are required. No immediate action is needed for consumers of core releases.']

## Technical details

The sync pipeline now fetches zipped Gutenberg assets directly from the GitHub Container Registry via `build-plugin-zip.yml`, replacing prior npm dependency resolution. In `package.json`, the `gutenberg.sha` property must be updated to immutable full-length commit hashes (tags and mutable refs are explicitly discouraged). During alpha, syncing occurs weekly post-release; at Beta 1+, trunk is pinned to the SHA of the corresponding `wp/X.Y` branch until SVN branching occurs. Post-branching, only critical fixes justify additional SHA bumps to avoid noisy CI artifacts. Build execution relies on `npm run build:dev` or the automated CI workflow. Trac ticketing is scoped per phase (separate tickets for each alpha hash bump; consolidated tickets for beta/RC syncs). Commit messages must follow the prescribed template, explicitly listing old/new hashes, cherry-picked PR references, and backport notes when applicable.

## Contribution

Authored by Jonathan Desrosiers on June 30, 2026, formalizing process corrections following infrastructure fixes in the 7.0 cycle (#64393, #65418) that restored successful syncs. The guidelines were peer-reviewed by core maintainers (@aduth, @youknowriad, @ellatrix, and others) to formalize cadence, prevent release branch contamination, and standardize tracking across the Gutenberg plugin and WordPress Core SVN repositories.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

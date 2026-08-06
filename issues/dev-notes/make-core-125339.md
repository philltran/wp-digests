# WordPress 7.1 Release Candidate Phase

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aaron Jorbin
- **Published:** 2026-08-05
- **Tags:** `General`, `7.1`
- **Link:** [https://make.wordpress.org/core/2026/08/05/wordpress-7-1-release-candidate-phase/](https://make.wordpress.org/core/2026/08/05/wordpress-7-1-release-candidate-phase/)
- **Usefulness:** 2/5

## Summary

WordPress 7.1 has entered the Release Candidate phase, triggering a set of release management policies that restrict trunk commits, enforce a hard string freeze, and limit milestone tickets to regressions and test suite expansions. Branching is temporarily delayed to allow GitHub Actions improvements to land, meaning all trunk commits now require double sign-off by core committers until the 7.1 branch is created.

## Impact

- **Core committers & contributors:** All trunk commits require double sign-off until the 7.1 branch is created. Use `dev-feedback` to request review and `dev-reviewed` to confirm approval. Test suite commits are exempt.
- **Plugin & theme developers:** No action required. The string freeze and ticket restrictions apply exclusively to WordPress Core development.
- **Polyglots & translators:** Translation branches will not be generated until the 7.1 branch is created, despite the hard string freeze being active.

## Technical details

The announcement enforces workflow constraints rather than code changes. Trac ticket workflow is updated to require the `dev-feedback` keyword to request a second committer review and the `dev-reviewed` keyword to mark approval. String modifications are locked behind the hard freeze, with exceptions only for critical UI text tagged with the `i18n-change` keyword. Ticket filtering restricts the `7.1` milestone to regressions and test suite expansions, allowing test commits to bypass code and string freezes.

## Contribution

The record carries no discussion detail; this notice was authored by Aaron Jorbin with peer review by @joedolson to coordinate standard RC-phase governance.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

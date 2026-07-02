# WordPress 7.0.1 RC1 is now available

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Estela Rueda
- **Published:** 2026-07-01
- **Tags:** `General`, `7-0-release-leads`, `7.0`, `7.0.1`, `minor releases`, `release`
- **Link:** [https://make.wordpress.org/core/2026/07/01/wordpress-7-0-1-rc1-is-now-available/](https://make.wordpress.org/core/2026/07/01/wordpress-7-0-1-rc1-is-now-available/)
- **Usefulness:** 3/5

## Summary

WordPress 7.0.1 RC1 is a bug-fix-only maintenance release for the 7.0 branch, targeting regressions and edge-case failures introduced during the 7.0 development cycle. The update resolves PHP 8.5 compatibility issues in `wp_get_attachment_image_src`, corrects CSS parsing and sanitization edge cases (`wp_kses`), restores a deprecated navigation block shim, and addresses several admin media UI and block editor rendering regressions. No new features or breaking changes are introduced.

## Impact

- **Plugin & Theme Developers**: Ensure PHP 8.5 compatibility if relying on `wp_get_attachment_image_src` (now corrected for proper array access). Developers referencing the legacy navigation submenu icon renderer will see `block_core_navigation_submenu_render_submenu_icon()` restored as a deprecated shim, preventing fatal errors in older block registrations.
- **Editor & Block Developers**: The `blockGap` fallback parser now correctly handles nested `var()` values. Navigation contexts for page lists are determined via block context rather than static checks, improving rendering accuracy in nested menu structures. Template part inspector restrictions on tabs and fills have been removed.
- **Admin & Site Owners**: Several visual regressions from the 7.0 admin reskin and media library are resolved (spinner misalignment, button crowding, scrollbar behavior). No action required beyond upgrading to RC1 or the final release; all changes are backward-compatible bug fixes.

## Technical details

Changes across Trac tickets (#63085–#65428) and Gutenberg PRs (#77530–#79691) are strictly scoped to correctness rather than architectural shifts. Key behavioral corrections include:

- **#64742**: Fixes an incorrect array access in `wp_get_attachment_image_src` that triggered warnings or undefined index errors under PHP 8.5.
- **#65270**: Resolves a sanitizer edge case where `wp_kses()` incorrectly corrupts valid CSS `background-image: url(...)` declarations, replacing malformed output with properly parsed styles.
- **#78484**: Restores `block_core_navigation_submenu_render_submenu_icon()` as a deprecated shim to maintain backwards compatibility for blocks that still reference the legacy renderer.
- **#77530 / #79181**: Updates block rendering to correctly parse nested `var()` values in `blockGap` fallback logic, removes restrictive tab/inspector fill constraints on Template Parts, and marks controlled mode block changes as non-persistent to prevent unwanted dirty states during passive renders.
- **#78493 / #79691**: Addresses UI hydration timing to eliminate black flashes on `wp-admin` pages before React hydration completes, and shifts focus management in the Visual Revisions slider to improve keyboard accessibility.

All modifications are confined to existing internal methods, media filter CSS adjustments, and block rendering pipelines. No public hooks, REST routes, or constants were added or removed.

## Contribution

The 7.0.1 maintenance branch was opened for RC1 testing on July 1, 2026, led by @jorbin, @cbravobernal, @estelaris, and @masteradhoc. The RC aggregates a curated set of Trac bug reports and merged Gutenberg pull requests introduced during the 7.0 cycle or intentionally deferred at its close. Changes were vetted under the double-committer sign-off workflow specific to the 7.0 branch, with the final release targeted for July 9, 2026 pending QA feedback.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

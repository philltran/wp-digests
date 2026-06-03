# Hotfix available for #65286

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aaron Jorbin
- **Published:** 2026-05-28
- **Tags:** `General`, `Classic Editor`, `hotfix`
- **Link:** [https://make.wordpress.org/core/2026/05/28/hotfix-available-for-65286/](https://make.wordpress.org/core/2026/05/28/hotfix-available-for-65286/)

## Summary

A targeted hotfix addresses Classic Editor plugin bug #65286, which broke the post publishing panel when users extended it with custom action buttons. The patch is distributed via plugin releases v1.7.0 and 1.4 to ensure immediate site stability. A permanent code-level resolution is scheduled for the WordPress 7.0.1 core maintenance release.

## Impact

- **Classic Editor plugin users** who have extended the post publishing panel with additional action buttons will experience broken behavior until patched.
- **Action required**: Update the `classic-editor` plugin to v1.7.0 or the dedicated hotfix build (v1.4).
- No breaking changes; this is a regression fix targeting panel interaction/rendering logic.

## Technical details

The upstream announcement does not provide granular file paths, function names, or unified diff details for #65286. The scope is explicitly limited to the **Classic Editor plugin** (`classic-editor`), specifically the logic handling extended post publishing panels and their attached action buttons. Plugin distributions v1.7.0 and 1.4 contain the patch, while a permanent fix targeting the WordPress core codebase is deferred to the 7.0.1 release cycle.

## Contribution

Bug #65286 was flagged for immediate hotfix publication due to its disruption of custom publishing panel extensions. Review and approval were conducted by @desrosj, @flixos90, @joemcgill, and @jeffpaul. The announcement and release coordination were published by Aaron Jorbin on Make WordPress Core on 2026-05-28. A core-side permanent fix was explicitly scoped for the upcoming 7.0.1 maintenance release rather than the hotfix cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

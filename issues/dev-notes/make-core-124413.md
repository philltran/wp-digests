# The Classic block stays in the inserter for WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Marin Atanasov
- **Published:** 2026-07-07
- **Tags:** `Core`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/07/the-classic-block-stays-in-the-inserter-for-wordpress-7-1/](https://make.wordpress.org/core/2026/07/07/the-classic-block-stays-in-the-inserter-for-wordpress-7-1/)
- **Usefulness:** 3/5

## Summary

The planned change to hide the Classic block from the default inserter in WordPress 7.1 has been reverted. The Classic block will continue to appear in the inserter, block library, and slash commands exactly as it does today. Consequently, the previously introduced `wp_classic_block_supports_inserter` filter was removed before shipping, eliminating any backward-compatibility footprint for developers to manage.

## Impact

- **Plugin & theme developers**: No behavioral change is required; the Classic block remains fully visible and functional by default. The `wp_classic_block_supports_inserter` filter does not exist in the upcoming release, so no migration from it is necessary.
- **Hosting & platform teams**: The companion "Enable Classic Block" plugin has been closed and can be safely deactivated or removed if previously installed.
- **Action required**: None. This revert restores prior default behavior and requires zero code or configuration adjustments.

## Technical details

- Removed the `wp_classic_block_supports_inserter` filter from the classic block registration/inserter logic. Because the feature never shipped in a stable release, the filter leaves no backward-compatibility shim or deprecation notice.
- Reinstated the Classic block's editing experience to its previous state, including the "Convert to blocks" toolbar action.
- Removed the block-level deprecation/migration notice that was temporarily added to the classic block (`core/freeform`).
- No REST schema, hook signature, or database change is introduced; this is a behavioral revert in the block editor frontend logic.

## Contribution

Author Marin Atanasov announced the reversal after gathering feedback that forcibly hiding the Classic block degraded the editor experience without directly advancing the goal of reducing TinyMCE dependencies. The decision was reviewed by @mamaduka and finalized during the WordPress 7.1 development cycle, redirecting the initiative toward improving "Convert to Blocks" fidelity, explicit TinyMCE dependency declarations, and on-demand asset loading rather than enforcing visibility changes through filters.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

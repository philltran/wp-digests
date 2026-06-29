# #79072: Navigation block: Fix responsive style states for typography settings

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Block] Navigation`, `[Feature] Style States`
- **Merged:** [`6ea36f1`](https://github.com/WordPress/gutenberg/commit/6ea36f1960359d3a383ec677b452f3ad26a3d662)
- **Discussion:** [#79072](https://github.com/WordPress/gutenberg/pull/79072) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes responsive typography styles in the Navigation block by copying generated state classes (`wp-states-*`) from the outer `<nav>` wrapper to inner `<ul class="wp-block-navigation__container">` elements. Previously, nested list containers intercepted font settings, causing breakpoint overrides (Desktop/Mobile/Tablet) to fail or leak across custom overlays.

## Impact

- **Theme & block developers**: Responsive typography presets now correctly cascade within navigation templates; no code changes required unless manually overriding container markup.
- **Agency/platform teams**: No migration steps needed; the corrected HTML structure ships automatically with the updated core version.
- **Note**: The implementation explicitly preserves existing state classes and isolates nested navigation blocks inside overlays to prevent unintended style inheritance.

## Technical details

- Introduces `block_core_navigation_add_state_class_to_container()` in `packages/block-library/src/navigation/index.php`.
- Registers a `render_block` filter at priority 11 with two arguments (`$block_content`, `$block`).
- Uses `WP_HTML_Tag_Processor` to iterate through the markup, extract the parent's `wp-states-[hex]` pattern via regex, and append it to matching `<ul>` nodes.
- Skips processing for non-navigation blocks, empty state classes, already-matched containers, and nested `.wp-block-navigation` elements inside overlays.
- Includes six PHPUnit tests in `phpunit/blocks/render-block-navigation-test.php` validating markup preservation, single/multiple container injection, and overlay isolation.

## Contribution

- Opened by @talldan to resolve typography breaks caused by complex responsive nav HTML structure (related to #62690).
- Co-authored by @tellthemachines and @andrewserong during review.
- Merged as `6ea36f1` following CI passes; reported flaky e2e failures were unrelated to this diff. Authors discussed optional test coverage for early-return edge cases but proceeded with the consolidated approach.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# Miscellaneous Editor Changes in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aki Hamano
- **Published:** 2026-08-04
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/08/04/miscellaneous-block-editor-changes-in-wordpress-7-1/](https://make.wordpress.org/core/2026/08/04/miscellaneous-block-editor-changes-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces a cluster of editor refinements that stabilize legacy APIs, fix client-side pagination behavior, and adjust CSS specificity for block-level presets. The Navigation block no longer forces font-size inheritance on child items, block transforms can now target specific variations, and the @wordpress/nux and @wordpress/reusable-blocks packages are deprecated in favor of modern alternatives. These changes improve layout predictability, simplify programmatic block manipulation, and clean up the client-side editor API surface.

## Impact

- **Plugin & theme developers:** The Navigation block stops applying font-size classes to core/navigation-link, core/navigation-submenu, core/page-list, and core/home-link. Use the render_block_core/navigation-link filter (and equivalents for other block names) to restore legacy propagation if your theme relies on it.
- **Block developers:** switchToBlockType() now accepts a third variationName argument, and block.json transforms support the variationName property. __experimentalCloneSanitizedBlock and __experimentalSanitizeBlockAttributes are deprecated; migrate to cloneSanitizedBlock and sanitizeBlockAttributes.
- **Core Data & headless consumers:** getEntityRecords() no longer artificially slices responses for non-paginated endpoints. If you previously relied on the per_page: -1 workaround, remove it. Custom entities backed by non-paginated REST routes must declare supportsPagination: false.
- **Theme authors (Global Styles):** Block-level preset classes now use :where() wrappers, dropping their CSS specificity to match root presets. Custom author CSS that previously relied on higher specificity to override block-level presets may need adjustment.
- **No action required:** The Markdown parser swap (showdown to marked) is internal to pasteHandler() and requires no code changes. The disableContentOnlyForTemplateParts editor setting defaults to false, preserving existing behavior.

## Technical details

- **Navigation font-size:** Removes explicit style application to child blocks, falling back to CSS inheritance. A PHP filter example using WP_HTML_Tag_Processor is provided to manually append has-{name}-font-size classes or inline styles to the target <UL>/<LI> elements.
- **Block transforms:** block.json transforms accept variationName. The JS function switchToBlockType( blocks, 'core/group', 'group-grid' ) now routes to the specified variation.
- **API stabilization:** @wordpress/blocks exports stable cloneSanitizedBlock and sanitizeBlockAttributes. The __experimental-prefixed versions now emit console deprecation warnings.
- **Markdown parsing:** pasteHandler() in @wordpress/blocks replaces showdown with marked. Output aligns with CommonMark/GFM; direct consumers of pasteHandler() should re-test edge cases.
- **Template parts editing:** Introduces disableContentOnlyForTemplateParts (boolean). Configurable via block_editor_settings_all filter in PHP or wp.data.dispatch( 'core/block-editor' ).updateSettings() in JS. Respects template-locked rendering mode.
- **Global Styles specificity:** Block-level preset classes (.has-*-color, .has-*-background-color, etc.) now wrap block selectors in :where(), reducing specificity from 0-1-1/0-2-0 to 0-1-0. This resolves tie-break conflicts with responsive state styles that both use !important.
- **Core Data pagination:** getEntityRecords() client-side slicing logic now checks supportsPagination. Non-paginated REST endpoints return full collections. Custom entities must explicitly set supportsPagination: false to prevent unintended pagination.
- **Package deprecations:** @wordpress/nux is stripped to a no-op compatibility shim. @wordpress/reusable-blocks components and data APIs emit deprecation warnings, directing developers to standard core entity methods for Synced Patterns.

## Contribution

The changes were compiled and published as a 7.1 dev note, aggregating multiple independent editor PRs and tickets merged during the beta cycle. Review credits go to @ramonopoly and @tyxla, with content contributions from @ellatrix, @isabel_brison, @mamaduka, @sarthaknagoshe2002, @talldanwp, and @0mirka00. The note reflects a coordinated cleanup of legacy editor packages and a series of targeted fixes for layout specificity, client-side data fetching, and block transform routing, with no major design debates or rejected alternatives documented in the release notes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

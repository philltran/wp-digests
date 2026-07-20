# #74042: Post editor: always iframe

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ellatrix
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `Needs Dev Note`, `[Package] Edit Post`, `[Package] E2E Tests`
- **Merged:** [`6fae0a0`](https://github.com/WordPress/gutenberg/commit/6fae0a0404cdaee8c576715f12ec56ac46180b26)
- **Discussion:** [#74042](https://github.com/WordPress/gutenberg/pull/74042) · 28 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The post editor now consistently renders in an iframe, removing the previous conditional fallback to a non-iframed canvas based on block `apiVersion`. This eliminates the layout and functionality inconsistencies that occurred when plugins mixed legacy (v1/v2) and modern (v3) blocks, and ensures all blocks execute in a controlled environment where compatibility issues can be identified early.

## Impact

- **Plugin & theme developers:** Blocks registered with `apiVersion` 1 or 2 will now run inside the iframe. If your block relies on direct DOM manipulation, global `window`/`document` access, or assumes a non-iframed layout, it may break. Test your blocks in the iframe environment.
- **E2E test authors:** The `switchToLegacyCanvas()` test utility is deprecated and will be removed in a future release. Tests calling it will emit a console warning.
- **Site owners & general users:** No action required. The editor will load more consistently across all block types.

## Technical details

The PR removes the `useShouldIframe` hook (`packages/edit-post/src/components/layout/use-should-iframe.js`) and strips the `disableIframe` prop from `EditorInterface` and `VisualEditor`. `VisualEditor` now hardcodes `shouldIframe={true}` when rendering `BlockCanvas`, and stops applying the `is-iframed` CSS class to `.edit-post-visual-editor`. The `MetaBoxesMain` component no longer accepts an `isLegacy` prop, and its conditional rendering logic is simplified. CSS rules previously scoped to `.edit-post-visual-editor.is-iframed` (such as `overflow: hidden`) are now applied unconditionally. The `switchToLegacyCanvas()` Playwright utility in `packages/e2e-test-utils-playwright/src/editor/switch-to-legacy-canvas.ts` now logs a deprecation warning. Documentation in `docs/reference-guides/block-api/block-api-versions/block-migration-for-iframe-editor-compatibility.md` is updated to reflect the permanent shift.

## Contribution

Opened and merged by @ellatrix with co-authors @t-hamano, @Mamaduka, @annezazu, @stokesman, @joedolson, @kubiqsk, and @USERSATOSHI. The change consolidates editor behavior by removing the conditional iframe logic that previously checked block `apiVersion` and device type. Reviewers noted the deprecation of `switchToLegacyCanvas()` and coordinated a follow-up effort to scan the WordPress.org plugin directory for legacy API blocks and notify maintainers.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

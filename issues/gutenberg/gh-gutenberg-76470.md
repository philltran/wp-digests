# #76470: Editor: clarify autosave failure notice

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @youknowriad
- **Labels:** `[Type] Enhancement`, `[Package] Editor`
- **Merged:** [`b89e094`](https://github.com/WordPress/gutenberg/commit/b89e0946e76dbbdb0eeab12c69af0d29dfcd58ef)
- **Discussion:** [#76470](https://github.com/WordPress/gutenberg/pull/76470) · 16 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg editor now displays dedicated, user-friendly messages for autosave and save failures, replacing generic fallback text with explicit reassurance about local persistence and retry behavior. The change also prevents raw, potentially cryptic server error messages from being appended directly to the notice UI. Instead, server errors are hidden behind a collapsible "Show details" disclosure, while client-generated errors (like offline or fetch failures) are omitted entirely to avoid redundancy.

## Impact

- **Plugin & theme developers:** No action required. This change is internal to the `@wordpress/editor` package and does not expose new public APIs, hooks, or filters.
- **Site owners & content editors:** Will see clearer, more actionable error notices when autosaves or manual saves fail, with server details hidden behind a disclosure to reduce confusion.
- **Headless & REST consumers:** No impact; this affects only the block editor UI layer and does not modify REST schema or client-side data stores.

## Technical details

The diff modifies `packages/editor/src/store/utils/notice-builder.js` to accept an `options` object in `getNotificationArgumentsForSaveFail`, introducing `AUTO_SAVE_FAILURE_NOTICE` and `AUTO_SAVE_OFFLINE_FAILURE_NOTICE` constants. When `options.isAutosave` is true, the builder returns these dedicated strings instead of the generic status-based messages. In `packages/editor/src/store/actions.js`, the `savePost` failure handler now imports `@wordpress/escape-html` and checks `error.code` against `CLIENT_GENERATED_ERROR_CODES` (`offline_error`, `fetch_error`). For non-client errors containing plain text, it constructs an HTML notice with `__unstableHTML: true` that wraps the escaped `error.message` in a `<details class="editor-save-error-details">` element. The handler also manually triggers `speak( noticeMessage, 'assertive' )` via `@wordpress/a11y` to ensure screen readers announce the notice before the raw HTML is rendered. A new SCSS file (`packages/editor/src/components/save-error-details/style.scss`) styles the disclosure, and `@wordpress/escape-html` is added to the editor package dependencies.

Before/after pattern in `notice-builder.js`:
```js
// Before
if ( error.message && ! /<\/?[^[>]*>/.test( error.message ) ) {
    noticeMessage = [ noticeMessage, error.message ].join( ' ' );
}

// After
if ( options?.isAutosave ) {
    noticeMessage = AUTO_SAVE_FAILURE_NOTICE;
}
```

## Contribution

The PR evolved through UX feedback emphasizing that error messages should reassure users and guide recovery rather than dump technical output. The author initially raised a concern about losing raw server messages entirely, but the final implementation resolves this by collapsing them into a `<details>` disclosure instead of stripping them or routing them to the console. A previous attempt to handle error messaging in a similar way was referenced during review, which informed the current approach. The merged diff reflects a focused refactor of the notice builder and save failure handler, with comprehensive unit tests added to cover both autosave and manual save paths.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

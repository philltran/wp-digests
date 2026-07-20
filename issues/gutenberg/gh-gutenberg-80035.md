# #80035: Style Book: Pass site editor settings to StyleBookPreview on the styles route

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Bug`, `[Package] Edit Site`, `[Feature] Style Book`
- **Merged:** [`036adfa`](https://github.com/WordPress/gutenberg/commit/036adfa1c18a79f725c08a01a75056e9cb66638c)
- **Discussion:** [#80035](https://github.com/WordPress/gutenberg/pull/80035) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `/styles` route in the Site Editor now passes the full `siteData.editorSettings` object to `StyleBookPreview`, aligning it with the existing `/stylebook` route. Previously, the styles route omitted the `settings` prop, causing the preview component to fall back to default editor settings and missing server-bootstrapped configuration like `__experimentalDiscussionSettings`. This fix prevents runtime errors and ensures consistent preview behavior across both routes.

## Impact

- **Site Editors & Theme/Plugin Developers:** No action required. The change resolves an internal inconsistency that could cause `undefined` errors when accessing experimental or server-bootstrapped settings in the Style Book preview.
- **Hosting & Platform Teams:** No configuration or migration changes needed.
- **Headless & REST Consumers:** Not applicable; this is strictly a client-side Site Editor routing fix.

## Technical details

The change modifies `packages/edit-site/src/components/site-editor-routes/styles.js`. The `StylesPreviewArea` component now accepts a `siteData` prop and explicitly passes `settings={ siteData.editorSettings }` to `<StyleBookPreview>`. The route definition updates the `preview` area to render `<StylesPreviewArea siteData={ siteData } />` instead of a bare `<StylesPreviewArea />`. This ensures the preview component receives the complete, server-bootstrapped editor settings object rather than falling back to `editorStore.getEditorSettings()`, which only contains `EDITOR_SETTINGS_DEFAULTS` when the full `<Editor>` context is absent.

## Contribution

Opened and merged by @Mamaduka (with co-author @t-hamano) to address root cause #73869. The fix was straightforward, aligning the `/styles` route with the existing `/stylebook` route implementation. Following the merge, @ramonjd identified a regression affecting live updates in the preview, which was subsequently resolved in a follow-up PR (#80459).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

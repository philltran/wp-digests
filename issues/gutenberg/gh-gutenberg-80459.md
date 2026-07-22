# #80459: Style Book: Restore live global styles updates on the styles route

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `Global Styles`, `[Package] Edit Site`, `Backported to WP Core`, `[Feature] Style Book`
- **Merged:** [`e2c5251`](https://github.com/WordPress/gutenberg/commit/e2c52519a5b9317910fa5e9cdbd03ada96422cb4)
- **Discussion:** [#80459](https://github.com/WordPress/gutenberg/pull/80459) · 6 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

The Style Book preview in the Site Editor now reflects unsaved global styles edits in real time. A regression introduced in #80035 caused the preview to freeze until a save or page reload, breaking the live-editing workflow for theme authors and site builders. This change restores immediate visual feedback when modifying global styles in the sidebar.

## Impact

- **Site editors & theme developers**: The Style Book preview now updates instantly as you adjust global styles, restoring the expected live-editing workflow.
- **Plugin & theme authors**: No breaking changes or API deprecations. The internal `StyleBookPreview` component now accepts an additional `userConfig` prop, but existing integrations remain unaffected.
- **Hosting & platform teams**: No configuration or migration steps required. The fix ships as a standard bug fix in the next WordPress release.

## Technical details

The diff modifies `packages/edit-site/src/components/site-editor-routes/styles.js` to import `useGlobalStyles` from `editorPrivateApis`. It extracts the live user configuration via `const { user: userConfig } = useGlobalStyles()` and passes it as the `userConfig` prop to `<StyleBookPreview>`. Previously, the preview only received static `siteData.editorSettings`, which caused it to ignore unsaved edits. The component logic was also refactored into `StyleBookPreviewArea` to cleanly isolate the hook call.

Before:
```jsx
<StyleBookPreview
  path={ section }
  onPathChange={ onChangeSection }
  settings={ siteData.editorSettings }
/>
```

After:
```jsx
const { user: userConfig } = useGlobalStyles();
// ...
<StyleBookPreview
  path={ section }
  onPathChange={ onChangeSection }
  settings={ siteData.editorSettings }
  userConfig={ userConfig }
/>
```

An e2e test was added to `test/e2e/specs/site-editor/style-book.spec.js` verifying that dispatching `editEntityRecord` to `core/globalStyles` immediately updates the preview iframe without a save.

## Contribution

The fix was authored by @ramonjd and co-authored with @andrewserong to address the regression. @t-hamano reviewed the patch and approved it for merge. Following the merge, the change was rapidly cherry-picked to both the `wp/7.1` and `release/23.6` branches to ensure inclusion in the upcoming release cycle. The record shows a straightforward review process with no rejected alternatives or extended design debates.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

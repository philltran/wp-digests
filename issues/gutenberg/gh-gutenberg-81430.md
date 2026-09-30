# #81430: DataViews: Move the rich text control into the editor package

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @youknowriad
- **Labels:** `[Type] Bug`, `[Package] Editor`, `[Package] DataViews`
- **Merged:** [`a91983c`](https://github.com/WordPress/gutenberg/commit/a91983ca6ccd04f42dcf994feea9c7d6dd602be3)
- **Discussion:** [#81430](https://github.com/WordPress/gutenberg/pull/81430) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The rich text control has been moved out of `@wordpress/dataviews` and into `@wordpress/editor`, next to the note form that is its only consumer. The built-in `Edit: { control: 'richtext' }` DataForm control, the `EditConfigRichText` type, and the package's `privateApis` export are removed, along with DataViews' dependency on `@wordpress/rich-text`. This fixes crashes in plugins that installed `@wordpress/dataviews` from npm, where a second copy of `@wordpress/rich-text` collided with the copy WordPress ships.

## Impact

**Plugin & theme developers**
- **Breaking (DataViews major bump):** `Edit: 'richtext'` / `control: 'richtext'` no longer resolves in DataForm. `EditConfigRichText` is gone from the `EditConfig` union, and `privateApis` is no longer exported from `@wordpress/dataviews`.
- A rich text field now needs a custom `Edit` component. The PR notes there is still no public way to assemble one outside `@wordpress/block-editor`; the remaining work is tracked as the `rich-text` item in #81230.
- Plugins that bundle `@wordpress/dataviews` from npm should no longer crash from the duplicate `@wordpress/rich-text` copy. Workarounds such as pinning WooCommerce to 17.1.0 or Jetpack's patch should no longer be needed.
- The PR describes the `richtext` control as having only worked inside Gutenberg's own build, so external breakage is expected to be limited.

**Site owners / hosting:** No action required.

**Core / Gutenberg:** A maintainer noted in review that DataViews isn't exposed through a global, so the change is not visible to WordPress 7.1 and no backport was needed.

**Bundle size:** The `/wp` DataViews build drops 122KB per the PR description, and the compressed-size report shows small reductions across the `block-editor`, `edit-site`, `editor` and `media-utils` bundles.

## Technical details

**Removed from `packages/dataviews`:**
- `src/components/dataform-controls/richtext/` (`index.tsx`, and the `control` implementation it imported), the `richtext` entry in `FORM_CONTROLS` in `dataform-controls/index.tsx`, and `test/richtext.tsx`.
- `src/private-apis.ts`, which locked `RichTextControl`, and the `export { privateApis }` line in `src/index.ts`.
- The `EditConfigRichText` type from `types/field-api.ts` and from the `EditConfig` union.
- The `.dataviews-controls__richtext` placeholder styles in `style.scss`.
- `@wordpress/rich-text` from `dependencies`, `@wordpress/format-library` from `devDependencies`, the matching `tsconfig.json` reference, and `modules.d.ts` (which only declared the untyped `@wordpress/format-library`).
- The `summary` rich text field from the `layout-regular` story.

**Added to `@wordpress/editor`:** per the description, the control, `FormatEdit`, `getAllowedFormats`, their tests and the placeholder styles move verbatim, with only the wrapper class name and import paths changed. The diff was truncated before the editor-side files, so those details come from the description and CHANGELOG entries, not the diff.

**Root cause (per the description):** the control unlocked `@wordpress/rich-text` at module scope, reachable from any `import { DataForm }` through both the control registry and `privateApis`. The `/wp` build then carried a second copy that reused core's `core/rich-text` store, so core's formats read the wrong React contexts and blanked the app. Moving the code means `@wordpress/rich-text` resolves to the same `wp.richText` the block editor uses: one registry, one store, one set of contexts.

The DataViews CHANGELOG gains a `Breaking Changes` entry; the editor CHANGELOG gains an `Internal` entry.

```ts
// Before (only worked inside Gutenberg's build)
{ id: 'summary', type: 'text', Edit: { control: 'richtext', allowedFormats: [ 'core/bold' ] } }

// After: supply a custom Edit component
{ id: 'summary', type: 'text', Edit: MyRichTextEdit }
```

## Contribution

Authored by @youknowriad with Claude Code, and it closes #81233 (plugins crashing against the bundled DataViews). @oandregal tested it and said DataViews may want to offer `richtext` as a proper control like `textarea`, but not through private API. He then prepared a 7.1 backport (#82186), and withdrew the need for it after @youknowriad asked why, since DataViews isn't available through the global.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

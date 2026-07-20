# #78471: DataViews: Add a richtext control backed by a private RichTextControl shell in @wordpress/components

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Status] In Progress`, `[Package] Components`, `[Package] Block library`, `[Package] Rich text`, `[Type] Feature`, `[Package] DataViews`
- **Merged:** [`07c0f12`](https://github.com/WordPress/gutenberg/commit/07c0f12e9b03a874e15e46299fbc894bc1ba6287)
- **Discussion:** [#78471](https://github.com/WordPress/gutenberg/pull/78471) · 40 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Introduces a new `richtext` DataForm control in `@wordpress/dataviews` that enables rich-text editing outside the block canvas. This capability is powered by a new private `ContentEditableControl` shell in `@wordpress/components` and shared event primitives (`RichTextShortcut`, `RichTextInputEvent`) moved to `@wordpress/rich-text`. The change decouples rich-text form fields from the `block-editor` module graph, allowing DataViews consumers to opt in declaratively with `Edit: 'richtext'`.

## Impact

- **Plugin & theme developers:** No action required. The `richtext` control is currently internal to `@wordpress/dataviews` and not yet applied to any existing fields.
- **DataViews consumers:** Can now declare rich-text fields in DataForm schemas using `Edit: 'richtext'`. Rich-text options (`allowedFormats`, `disableFormats`, `preserveWhiteSpace`, etc.) pass through the field `config`.
- **Platform & hosting teams:** No configuration or migration steps needed. The `ContentEditableControl` component is registered as a private API in `@wordpress/components` and is not intended for external consumption.

## Technical details

- **`@wordpress/rich-text`**: `RichTextShortcut` and `RichTextInputEvent` are moved from `block-editor` (which re-exports them unchanged). Private `shortcutsListener` and `inputEventsListener` helpers are added for event dispatch.
- **`@wordpress/components`**: Adds a private, presentational `ContentEditableControl` (`packages/components/src/content-editable-control/index.tsx`). It renders a `BaseControl`-wrapped `div` with `role="textbox"`, `contentEditable`, and selection/blur handling delegated to the consumer. The component and its validated variant (`ValidatedContentEditableControl`) are locked in `packages/components/src/private-apis.ts`.
- **`@wordpress/dataviews`**: Registers a new `richtext` control in the `FORM_CONTROLS` registry (`packages/dataviews/src/components/dataform-controls/richtext/`). It assembles `useRichText`, `FormatEdit`, shortcut/input contexts, and Enter handling on top of the shell. Fields opt in via `Edit: 'richtext'`, with rich-text options passed through the field `config`.
- **Usage pattern change:**
  ```diff
  // Before: no rich-text support in DataViews
  { key: 'caption', Edit: 'text' }

  // After: opt-in rich-text control
  { key: 'caption', Edit: 'richtext', config: { allowedFormats: [ 'core/bold', 'core/italic' ] } }
  ```

## Contribution

Opened by @adamsilverstein and merged in commit `07c0f12`. The PR consolidated review feedback from @talldan, @ciampo, and @andrewserong to avoid creating a standalone `@wordpress/rich-text-control` package, instead placing a presentational shell in `@wordpress/components` and keeping the rich-text assembly in `@wordpress/dataviews`. Shared primitives were extracted to trunk in #79828 and merged first to ensure clean integration. Co-authored by @talldan, @mirka, @andrewserong, @ntsekouras, @Mamaduka, @ciampo, @t-hamano, @ellatrix, @mcsf, @dmsnell, and @westonruter.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

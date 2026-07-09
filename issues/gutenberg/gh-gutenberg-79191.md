# #79191: Block Editor: Allow overriding `disableContentOnlyForTemplateParts` setting

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @CookieDarb
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block editor`, `Needs Dev Note`
- **Merged:** [`7bcc471`](https://github.com/WordPress/gutenberg/commit/7bcc471d813f286daa1cd67d3106c7669b58e6f1)
- **Discussion:** [#79191](https://github.com/WordPress/gutenberg/pull/79191) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Block Editor now correctly respects the `disableContentOnlyForTemplateParts` setting when passed via the `block_editor_settings_all` PHP filter. Previously, this setting was silently ignored during editor initialization, preventing theme developers from overriding WordPress 7.0’s `contentOnly` lock on Template Parts to restore standard inner-block editing.

## Impact

- **Theme & plugin developers**: Can now successfully pass `disableContentOnlyForTemplateParts => true` via the `block_editor_settings_all` filter to unlock editing inside `core/template-part` blocks.
- **No action required** for existing installs; this resolves a silent drop of a documented filter parameter without altering default behavior or introducing breaking changes.

## Technical details

Updates `packages/editor/src/components/provider/use-block-editor-settings.js` to wire the setting through the editor settings store.
1. Added `'disableContentOnlyForTemplateParts'` to the `BLOCK_EDITOR_SETTINGS` registry array.
2. Modified the resolved assignment in `useBlockEditorSettings` from:
   ```js
   renderingMode === 'template-locked',
   ```
   To:
   ```js
   renderingMode === 'template-locked' || settings.disableContentOnlyForTemplateParts,
   ```
3. Expanded unit tests in `packages/block-editor/src/store/test/private-selectors.js` and `reducer.js` to verify that `isSectionBlock` and `derivedBlockEditingModes` correctly reflect the bypass for `core/template-part` blocks when the setting is enabled.

## Contribution

Opened by @CookieDarb (closes #79189) and refined through review by @ramonjd and @talldan. The PR merged as commit `7bcc471`, resolving a documented limitation where the block editor settings filter was dropping the setting during initialization.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

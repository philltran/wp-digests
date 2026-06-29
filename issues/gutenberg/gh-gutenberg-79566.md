# #79566: Commands: Suggest pattern editing toggle for selected patterns

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Feature] Patterns`
- **Merged:** [`c8d133f`](https://github.com/WordPress/gutenberg/commit/c8d133fbde9dd86a2f2f42d2bce8d1fb7d2060d5)
- **Discussion:** [#79566](https://github.com/WordPress/gutenberg/pull/79566) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The block editor's command palette now automatically suggests the pattern editing toggle when a pattern or template part block is selected, or when full pattern editing mode is already active. Previously, this toggle was only discoverable through manual search queries or the Recent commands list, reducing visibility during pattern-heavy editing workflows.

## Impact

- **Block editor users & theme/plugin developers**: The "Enable/Disable editing all patterns" command now surfaces automatically in Suggestions when working inside a pattern or template part, improving workflow ergonomics without requiring keyboard searches.
- **Plugin & theme developers**: No code changes required. This is an internal editor UX enhancement that does not expose new public APIs, modify settings schemas, or break existing block/pattern implementations.
- **Headless/REST consumers**: Unaffected; this change operates exclusively within the client-side command palette subsystem.

## Technical details

- Extracts the existing `core/toggle-pattern-editing` command into a reusable internal helper `getTogglePatternEditingCommand()` that accepts editor settings and dispatch functions as options.
- Registers a new contextual loader via `useCommandLoader()` bound to the internal `'block-selection-edit'` context.
- The loader evaluates block selection state using `blockEditorStore` (`getBlockName`, `getBlockAttributes`, `getBlockParents`) and checks for pattern/template part ancestry via `isPatternOrTemplatePartBlock(blockName, attributes)`, which matches `attributes?.metadata?.patternName` or `blockName === 'core/template-part'`.
- Conditionally returns the toggle command when `disableContentOnlyForPatternsAndTemplateParts` is false (or when selection contains a pattern), while suppressing suggestions during preview mode or when outside pattern contexts.
- Adds E2E assertions in `test/e2e/specs/editor/various/patterns.spec.js` to verify suggestion visibility before and after enabling full pattern editing.

## Contribution

Opened by @ramonjd as a follow-up to #78383, the PR was reviewed and merged with co-authorship from @andrewserong. The diff reflects a straightforward refactor extracting the toggle logic into a helper and wiring it to the `block-selection-edit` context, accompanied by targeted E2E test coverage. No alternative approaches were debated in the upstream discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

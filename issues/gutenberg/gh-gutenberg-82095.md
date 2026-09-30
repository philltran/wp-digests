# #82095: UI: Add Field.VisualLabel

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Tool] ESLint plugin`, `[Package] UI`
- **Merged:** [`c53d375`](https://github.com/WordPress/gutenberg/commit/c53d3752f8bb49dc439c421b6aed38055836f458)
- **Discussion:** [#82095](https://github.com/WordPress/gutenberg/pull/82095) · 4 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

`@wordpress/ui` gains `Field.VisualLabel`, a purely visual label styled identically to `Field.Label` that can be rendered outside `Field.Root` and carries no label semantics. It is the `@wordpress/ui` counterpart to `BaseControl.VisualLabel`, for controls that are already accessibly labeled but need a label for layout. The same PR marks `BaseControl` as not recommended in Storybook and adds it to the `@wordpress/use-recommended-components` ESLint denylist.

## Impact

**Plugin & theme developers**
- New public API: `Field.VisualLabel` in `@wordpress/ui`. Use it when a control already has an accessible name and only needs a visual label. It is not associated with any control.
- For group legends, use `Fieldset` / `Fieldset.Legend`. For descriptions, use `Field.Description` / `Fieldset.Description` so they are associated. Do not use `Field.VisualLabel` as a substitute.
- `BaseControl` is now flagged `not-recommended` (previously `recommended`) in Storybook.

**ESLint / tooling**
- `@wordpress/eslint-plugin`'s `use-recommended-components` rule now reports `BaseControl` imported from `@wordpress/components`, pointing to `Field`, `Field.VisualLabel`, and `Fieldset`.
- Projects using this rule will see new lint findings on existing `BaseControl` usage after upgrading the plugin. Gutenberg itself absorbed this by adding entries to `tools/eslint/suppressions.json`.
- `BaseControl` is not removed or deprecated at runtime; no runtime behavior change.

## Technical details

**New component:** `packages/ui/src/form/primitives/field/visual-label.tsx` exports `VisualLabel`, a `forwardRef<HTMLSpanElement, FieldVisualLabelProps>` built on Base UI's `useRender` and `mergeProps`. The default tag is `span`. Class names combine `fieldStyles.label` (from `utils/css/field.module.css`), an optional `fieldStyles[`is-${variant}`]`, and the caller's `className`. It has `displayName = 'Field.VisualLabel'` and is exported from `field/index.ts`.

**Types (`field/types.ts`):** the `variant?: 'default' | 'plain'` prop is extracted into a shared `FieldLabelVariantProps`, used by both `FieldLabelProps` and the new `FieldVisualLabelProps` (`ComponentProps<'span'>` plus `variant` and `children`). Props listed in the manifest: `children`, `className`, `render`, `style`, `variant`.

```tsx
<Stack direction="column" gap="sm" align="flex-start">
	<Field.VisualLabel>Author</Field.VisualLabel>
	<Button variant="outline">Select an author</Button>
</Stack>
```

**Other changes:**
- Adds a ref-forwarding jsdom test, a `WithVisualLabel` story, and overview guidance in `form/primitives/stories/overview.mdx`.
- Updates `storybook/components-manifest.yml`.
- Adds a `BaseControl` message to the `DENYLIST` in `packages/eslint-plugin/rules/use-recommended-components.js`.
- Adds CHANGELOG entries for `@wordpress/ui` (New Features) and `@wordpress/eslint-plugin` (Enhancements).
- `tools/eslint/suppressions.json` has count bumps and new entries for existing `BaseControl` usages in `block-editor`, `block-library`, `dataviews`, `editor`, `fields`, `global-styles-ui`, and `patterns`.

No hooks, REST, or DB changes.

## Contribution

Follow-up to #80636. In review, a question was raised about making this a `Text` variant instead, and a concern that putting a visual label in `Field` legitimizes a pattern that may not be desirable. @mirka argued `Text` only covers typography and this needs a color locked to the label color. She also cited an audit of 21 existing `BaseControl.VisualLabel` call sites: 9 fit the `Fieldset` pattern, 3 were accessibility bugs (orphan legend or group label), and 9 were valid visual-label uses. She chose to start with `Field.VisualLabel` and add guardrails if antipatterns appear. `Field.VisualDescription` was deliberately left out.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #81359: UI: Add TextareaControl component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] UI`
- **Merged:** [`94a108d`](https://github.com/WordPress/gutenberg/commit/94a108d3c936b63237ac63de40d2e365e0d983d9)
- **Discussion:** [#81359](https://github.com/WordPress/gutenberg/pull/81359) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a `TextareaControl` component to `@wordpress/ui`, composed from the existing `Field` and `Textarea` primitives, mirroring the existing `InputControl` pattern. The legacy `TextareaControl` in `@wordpress/components` is now marked `not-recommended` in Storybook, and the `@wordpress/use-recommended-components` ESLint rule flags it with a pointer to a new migration guide. The `Textarea` primitive is also promoted from `use-with-caution` to `recommended`.

## Impact

**Plugin & theme developers**
- `@wordpress/ui` now exports `TextareaControl` as a public component. It is the intended replacement for `TextareaControl` from `@wordpress/components`.
- The legacy component is not deprecated at runtime; the change is limited to Storybook status metadata and lint guidance.
- Code that imports `TextareaControl` from `@wordpress/components` will now trigger the `@wordpress/use-recommended-components` rule. Gutenberg suppresses existing violations in `tools/eslint/suppressions.json`. Other projects using the rule should expect new warnings or errors on upgrade.
- Migration is not a drop-in swap: `onChange` becomes `onValueChange`, `help` becomes `description` (plain text) or `details` (markup), and `label` must be a plain `string`.

**Site owners / hosting / headless consumers**
- No action required.

## Technical details

**New component** (`packages/ui/src/form/textarea-control/`), exported from `packages/ui/src/form/index.ts`:

```tsx
<Field.Root className={ className }>
  <Field.Label hideFromVision={ hideLabelFromVision }>{ label }</Field.Label>
  <Textarea ref={ ref } { ...restProps } />
  { description && <Field.Description>{ description }</Field.Description> }
  { details && <Field.Details>{ details }</Field.Details> }
</Field.Root>
```

- It is a `forwardRef<HTMLTextAreaElement>` component. `TextareaControlProps` is `React.ComponentProps<typeof Textarea> & ControlProps`.
- The manifest lists its props as `className`, `defaultValue`, `description`, `details`, `disabled`, `hideLabelFromVision`, `label`, `onValueChange`, `render`, `rows`, `style`, and `value`.
- Unit tests cover ref forwarding, visible and hidden labels, description, and details.

**Prop mapping from the legacy component** (from the new migration guide):

```jsx
// Before
<TextareaControl label="Description" value={ value } onChange={ setValue } help="..." />
// After (@wordpress/ui)
<TextareaControl label="Description" value={ value } onValueChange={ setValue } description="..." />
```

`hideLabelFromVision` and `rows` (default `4`) are unchanged. For non-string labels, the guide says to compose `Field.Root`, `Field.Label`, and `Textarea` directly.

**ESLint rule** (`packages/eslint-plugin/rules/use-recommended-components.js`):
- `Textarea` and `TextareaControl` are added to the `@wordpress/ui` `ALLOWLIST`.
- `TextareaControl` is added to the `DENYLIST` for `@wordpress/components`, with the message "Use `TextareaControl` from `@wordpress/ui` instead."
- The rule docs link to the new migration guide.

**Other changes:**
- The `Textarea` primitive's Storybook status goes from `use-with-caution` to `recommended`, and it gets a JSDoc note to prefer `TextareaControl` for standard label and description use.
- Both `Textarea` and the new `TextareaControl` stories add the `manifest` tag and an explicit `render` function as a workaround for storybookjs/storybook#34877.
- The legacy `TextareaControl` story drops the `manifest` tag.
- `storybook/components-manifest.yml` is regenerated.
- A stray `htmlFor`/`id` is removed from the `InputControl` migration guide example.
- `packages/ui/CHANGELOG.md` gets an Enhancements entry.
- `tools/eslint/suppressions.json` gains or increments suppression counts for existing `TextareaControl` usages (for example in `block-library` and `editor`).

## Contribution

The PR was authored by @mirka as a prerequisite for #81230, which removes private `@wordpress/components` API usage from DataViews and needs a public textarea control before a `ValidatedTextareaControl` can be built in wp-ui. In review, @ciampo asked whether the Storybook examples should have accessible labels and suggested more detailed ESLint replacement guidance (`onChange` to `onValueChange`, `help` to `description`/`details`). @mirka deferred the accessible-label work to a separate PR (#81777) and added a migration guide modeled on the `InputControl` one.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

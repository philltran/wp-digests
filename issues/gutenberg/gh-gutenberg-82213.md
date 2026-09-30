# #82213: UI: Add CheckboxControl

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `[Package] UI`
- **Merged:** [`7134412`](https://github.com/WordPress/gutenberg/commit/713441267b487394e58f4d98c43403722c8d479e)
- **Discussion:** [#82213](https://github.com/WordPress/gutenberg/pull/82213) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a labeled `CheckboxControl` to `@wordpress/ui`, composed from the existing `Checkbox` and `Field` primitives. It gives the package a boolean-field counterpart to its text, textarea and select controls, with built-in label, description, details, a visually hidden label option, and first-line alignment for wrapping labels. The PR also fixes `Field.Label variant="plain"` so it resets to the default font weight.

## Impact

**Plugin & theme developers (using `@wordpress/ui`)**
- New public export `CheckboxControl`, so you no longer need to hand-compose `Field.Root`, `Field.Item`, `Checkbox` and `Field.Label`.
- Visual fix: `Field.Label` with `variant="plain"` now uses `--wpds-typography-font-weight-default` rather than inheriting the heavier label weight. Anyone relying on plain labels rendering heavier will see a change.

**Everyone else**
- No action required. The `@wordpress/components` `CheckboxControl` remains the recommended control, and this PR does not mark the new one as recommended. The story notes in `@wordpress/components` now link to the new Storybook docs page.

## Technical details

**New files under `packages/ui/src/form/checkbox-control/`:** `checkbox-control.tsx`, `types.ts`, `style.module.css`, `index.ts`, a story and a jsdom test. It is re-exported via `packages/ui/src/form/index.ts` (`export * from './checkbox-control'`).

**Component:** a `forwardRef<HTMLSpanElement, CheckboxControlProps>` that renders `Field.Root` (receiving `name` and `className`) containing a `Field.Item` with `render={<Stack gap="sm" align="start" />}`. Inside are a `checkbox-wrapper` div holding `Checkbox` and a text container with `Field.Label variant="plain"`. `label`, `description`, `details` and `hideLabelFromVision` are destructured. The remaining props and the `ref` go to `Checkbox`. When `description` or `details` is present, the text container is a column `Stack` with `gap="xs"`; otherwise it is a `Fragment`. `Field.Description` and `Field.Details` render only when provided.

**Types:** `CheckboxControlProps = React.ComponentProps<typeof Checkbox> & ControlProps`.

**Alignment CSS** (in `@layer wp-ui`): the wrapper is `display: flex; align-items: center; height: 1lh`, so the checkbox centers on the first label line. A private custom property `--_wp-ui-checkbox-control-label-line-height` is `max(16px, var(--wpds-typography-line-height-xs))`. It is fed to `--wp-ui-field-label-line-height` on the label, and the checkbox size is set via `--wp-ui-checkbox-input-size`. The label gets `cursor: var(--wpds-cursor-control)` unless the checkbox has `[data-disabled]`.

**Field.Label fix** in `utils/css/field.module.css`:

```css
&.is-plain {
  font-size: var(--wpds-typography-font-size-md);
  font-weight: var(--wpds-typography-font-weight-default); /* added */
  text-transform: none;
}
```

**Tests** cover ref forwarding, accessible name and description, label-click toggling, and form data: a custom `name` and `value`, an unchecked box being omitted, the default `on` value, and multiple same-named checkboxes yielding `['a', 'c']`. The CHANGELOG records both the new feature and the bug fix.

## Contribution

Authored by @mirka, with @ciampo credited in the props list. The recorded discussion contains only bot output (bundle size, flaky e2e report, CodeRabbit), and CodeRabbit skipped its automated review, so there is no visible design debate. One explicit scoping decision is that the new control is deliberately not marked as recommended over the `@wordpress/components` one.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

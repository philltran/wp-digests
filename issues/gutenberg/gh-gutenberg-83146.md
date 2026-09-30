# #83146: UI: Add SwitchControl component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Feature] UI Components`, `[Type] Feature`, `[Package] UI`
- **Merged:** [`554f279`](https://github.com/WordPress/gutenberg/commit/554f279e23c7b200d00808e5e2c621b53662425f)
- **Discussion:** [#83146](https://github.com/WordPress/gutenberg/pull/83146) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a `SwitchControl` component to `@wordpress/ui`, a labeled wrapper around the existing `Switch` form primitive that composes `Field.Root`, `Field.Label`, `Field.Description`, and `Field.Details` into a single row layout. It follows the same pattern as the existing `CheckboxControl` and `InputControl` components, filling the gap for on/off settings that need a labeled switch. The component is marked `use-with-caution` in Storybook, pending a style-consistency review against `@wordpress/components`.

## Impact

- **Plugin & theme developers using `@wordpress/ui`:** A new `SwitchControl` export is available from the form module. It is explicitly flagged `use-with-caution` — not yet recommended for use alongside `@wordpress/components` components. No production adoption is advised until the consistency review (tracked in WordPress/gutenberg#76135) is complete.
- **Existing `@wordpress/components` users:** No change. `SwitchControl` lives in the separate `@wordpress/ui` package and does not touch `@wordpress/components`.
- **No action required** for any existing code. This is a purely additive export.

## Technical details

The new component lives in `packages/ui/src/form/switch-control/switch-control.tsx` and is exported via `packages/ui/src/form/index.ts`.

**Props type** (`types.ts`):
```ts
export type SwitchControlProps = React.ComponentProps< typeof Switch > &
	ControlProps;
```
This means it accepts all `Switch` primitive props (`checked`, `defaultChecked`, `onCheckedChange`, `disabled`, `value`, etc.) plus the shared `ControlProps` (`label`, `description`, `details`, `hideLabelFromVision`, `name`, `className`).

**Render structure:** `Field.Root` is rendered as `<Stack gap="sm" align="start" />` (a horizontal row). Inside, a `.switch-wrapper` div holds the `Switch` primitive, and a `TextContainer` (a `Stack direction="column" gap="xs"` when description or details are present, otherwise a `Fragment`) holds `Field.Label`, `Field.Description`, and `Field.Details`.

**CSS** (`style.module.css`): The switch wrapper uses `height: 1lh` and `line-height: var(--_wp-ui-switch-control-label-line-height)` to vertically center the switch on the first line of a potentially multi-line label. The switch height is set to `var(--wpds-dimension-size-2xs)`. A `:not(:has(.switch[data-disabled]))` selector applies a pointer cursor to the label only when the switch is enabled.

**Form data behavior** (verified by tests): An unchecked switch omits its `name` from `FormData`; a checked switch submits its `value` prop (defaulting to `"on"`).

**JSDoc update:** The `Switch` primitive in `packages/ui/src/form/primitives/switch/switch.tsx` gained the line `* Prefer SwitchControl for labeled items.`

## Contribution

Opened by @mirka with @simison as co-author. The PR carries only 2 comments and 0 reactions — no visible design debate or alternative approaches in the provided record. The `use-with-caution` status and the pointer to issue #76135 indicate the component is part of a broader `@wordpress/ui` readiness effort rather than a standalone feature decision.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

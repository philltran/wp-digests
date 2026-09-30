# #83143: UI: Add RadioGroupControl

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`c631bea`](https://github.com/WordPress/gutenberg/commit/c631beacf0271ff13010ec59693b2b67793f8339)
- **Discussion:** [#83143](https://github.com/WordPress/gutenberg/pull/83143) · 2 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Adds `RadioGroupControl` to `@wordpress/ui`, a labeled radio-group component that renders from an `items` array. It composes the existing `Fieldset`, `Field`, `Radio`, and `RadioGroup` primitives into a single drop-in control, matching the pattern already established by `CheckboxControl`, `SelectControl`, and `SwitchControl`. This closes the gap where developers had to manually wire up `Fieldset` + `Field` + per-item labels for a standard mutually-exclusive option list.

## Impact

- **Plugin & theme developers (editor/admin UI):** New import available from `@wordpress/ui`: `import { RadioGroupControl } from '@wordpress/ui'`. Use it in block inspector panels, settings screens, or any React-based admin UI where you previously composed `Fieldset` + `Field` + `Radio` by hand. No migration required for existing code.
- **Site owners / hosting / headless consumers:** No action required. This is a client-side React component with no REST, DB, or template impact.

## Technical details

The component lives at `packages/ui/src/form/radio-group-control/radio-group-control.tsx` and is exported from `packages/ui/src/form/index.ts`.

**Structure:** `RadioGroupControl` renders a `Field.Root` (which carries the `name` attribute for form submission) wrapping a `Fieldset.Root` whose `render` prop is a `RadioGroup`. Inside the fieldset:

- `Fieldset.Legend` (with `hideFromVision` bound to `hideLabelFromVision`) renders the group label.
- `Fieldset.Description` and `Fieldset.Details` render under the legend, before the radio items.
- Each entry in `items` becomes a `Field.Item` containing a `Radio` (with `value` and `disabled`), a `Field.Label` (variant `"plain"`), and an optional `Field.Description`.

**Props type** (`RadioGroupControlProps` in `types.ts`):

```ts
Omit<React.ComponentProps<typeof RadioGroup>, 'children'> &
  ControlProps & {
    items: {
      label: string;
      value: string;
      description?: string;
      disabled?: boolean;
    }[];
  }
```

This inherits `value`, `defaultValue`, `onValueChange`, `disabled`, and `name` from the `RadioGroup` primitive, plus `label`, `description`, `details`, and `hideLabelFromVision` from the shared `ControlProps`.

**Styling** (`style.module.css`): Uses CSS custom properties `--_wp-ui-radio-group-control-radio-height` (16 px) and `--_wp-ui-radio-group-control-label-line-height` to keep the radio vertically centered on the first line of a wrapped label. A `:has(.radio[data-disabled])` selector suppresses the pointer cursor on disabled items.

**Form-data parity:** The jsdom test suite includes a test that submits a form containing `RadioGroupControl` alongside a native `<fieldset>`/`<input type="radio">` group and asserts `FormData` output is identical, confirming the Base UI `RadioGroup` integration preserves native form semantics.

Bundle impact: −26 B net (0%).

## Contribution

Authored by @mirka with @simison as co-author, closing long-standing issue #63734. The visible discussion consists only of the automated bundle-size and co-author bot comments; no design debate or alternative approaches are recorded in the PR thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

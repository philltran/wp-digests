# #83039: UI: Add Switch form primitive

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`5ff63f3`](https://github.com/WordPress/gutenberg/commit/5ff63f3fd373eb2460518f898ccefca89de23a3b)
- **Discussion:** [#83039](https://github.com/WordPress/gutenberg/pull/83039) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds a `Switch` form primitive to `@wordpress/ui`, wrapping Base UI's switch component and styling it with WordPress Design System tokens. It is a low-level, unlabelled control — the PR notes that a labelled `SwitchControl` will follow in a separate change. The component is part of the broader form-primitives effort tracked in #74178 and is exported from the package's form-primitives barrel.

## Impact

- **Plugin & theme developers building on `@wordpress/ui`:** New `Switch` export available from `@wordpress/ui`. Marked **use-with-caution** in Storybook pending a style-consistency review against `@wordpress/components` (see #76135). Not yet recommended for production use alongside existing `@wordpress/components` controls.
- **No breaking changes.** No existing API is modified or removed.
- **No action required** for existing code. The labelled `SwitchControl` that most consumers would actually render is explicitly deferred to a follow-up PR.

## Technical details

New component at `packages/ui/src/form/primitives/switch/switch.tsx`:

```tsx
import { Switch as _Switch } from '@base-ui/react/switch';

export const Switch = forwardRef<HTMLSpanElement, SwitchProps>(
  function Switch({ className, ...props }, ref) {
    return (
      <_Switch.Root ref={ref} className={clsx(resetStyles['box-sizing'], styles.root, className)} {...props}>
        <_Switch.Thumb className={styles.thumb} />
      </_Switch.Root>
    );
  }
);
```

- **Props type** (`types.ts`): `SwitchProps = ComponentProps<typeof _Switch.Root>`, so all Base UI switch props (`defaultChecked`, `checked`, `disabled`, `onValueChange`, ARIA attributes, etc.) pass through.
- **CSS** (`style.module.scss`): 16 px track height via `--wpds-dimension-size-2xs`; checked state uses `--wpds-color-background-interactive-brand-strong`; thumb translates via `transform: translateX(var(--_wp-ui-switch-height))` with an RTL override (`:dir(rtl)` negates the offset). Forced-colors mode maps track to `Highlight`/`GrayText` and thumb to `HighlightText`/`Canvas`/`ButtonText`/`GrayText` depending on state.
- **Export** added to `packages/ui/src/form/primitives/index.ts` as `export { Switch } from './switch'`.
- **Storybook** stories: `Default`, `Checked`, `Disabled`, `DisabledChecked` under `Design System/Components/Form/Primitives/Switch`.
- **JSDOM tests** verify ref forwarding to an `HTMLSpanElement` and that `defaultChecked` produces a checked `role="switch"` element.
- Minor: a comment added to `packages/ui/src/utils/css/focus.module.scss` noting that the shared focus-ring utility classes are legacy and new code should use the `mixins.focus-ring()` mixin directly.

## Contribution

Opened by @mirka as part of the #74178 form-primitives track. During review, @aduth flagged that the "Disabled Checked" state was visually indistinguishable in forced-colors mode (both track and thumb rendered `GrayText`). @mirka fixed it by switching the disabled-checked thumb to `Canvas` so the knob remains visible against the `GrayText` track. The PR merged with co-authorship from both contributors; no alternative approaches or reverts are recorded in the four comments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

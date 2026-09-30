# #82259: ToggleGroupControl: Honor the root disabled prop

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Bug`, `[Package] Components`
- **Merged:** [`c18c35d`](https://github.com/WordPress/gutenberg/commit/c18c35d190f82d48c0f80ab7692a564145ddda3a)
- **Discussion:** [#82259](https://github.com/WordPress/gutenberg/pull/82259) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`ToggleGroupControl` now honors its root `disabled` prop across both variants. Previously the radio-group variant disabled its options only as a side effect of passing `disabled` to `Ariakit.RadioGroup`, while the `isDeselectable` button-group variant forwarded it to a wrapper `div`, leaving the buttons clickable. Both variants now disable every option, expose `aria-disabled` on the group, and get consistent disabled styling (faded options and selected indicator, no enclosing-border hover).

## Impact

- **Plugin & theme developers using `ToggleGroupControl` from `@wordpress/components`:**
  - Passing `disabled` on the root now reliably makes the whole control unselectable, including with `isDeselectable`. A pressed option can no longer be deselected while the group is disabled.
  - Disabled options are skipped in tab/arrow navigation. `onChange` is not called on click or arrow-key input, and the selected value is preserved.
  - The disabled state now has a visual treatment, so any UI that already passed `disabled` (e.g. the DataViews "Items per page" control) will look different in the deselectable variant, where it now also actually blocks interaction.
  - The option-level `disabled` prop on `ToggleGroupControlOption` and `ToggleGroupControlOptionIcon` is now documented and named in the public option types.
- **Not covered:** `focusableWhenDisabled` is explicitly out of scope, so disabled options are not focusable.
- No migration is needed. Code that relied on `disabled` being ignored on the deselectable variant will now see it take effect.

## Technical details

The change is in `packages/components/src/toggle-group-control/`.

- `toggle-group-control/component.tsx`: `disabled` (default `false`) is destructured out of the props before DOM spreading and passed explicitly to `MainControl`.
- `toggle-group-control/as-radio-group.tsx`: `disabled` is added to the context value as `Boolean( disabled )` and passed to `Ariakit.RadioGroup` as `disabled={ disabled || undefined }`. Ariakit then sets `aria-disabled` on the group and disables the descendant radios.
- `toggle-group-control/as-button-group.tsx`: `disabled` is added to the context value and set on the group `View` as `aria-disabled={ disabled || undefined }`. It is no longer leaked onto the `div` as a native `disabled` attribute.
- `toggle-group-control-option-base/component.tsx`: computes `isOptionDisabled = Boolean( toggleGroupControlContext.disabled || disabled )` and passes it as the native `disabled` on the `isDeselectable` `<button>`.
- `toggle-group-control/style.module.scss`: adds an `&[aria-disabled="true"]` block that reduces the selected-indicator `::before` opacity, and per the PR description also fades options and suppresses the enclosing-border hover. The diff is truncated, so exact values aren't shown.
- README updates for the group, option, and option-icon components document `disabled`. A `Disabled` Storybook story was added, and the story template now seeds its state from `props.value`. `CHANGELOG.md` gets an Enhancements entry.
- Tests in `test/index.jsdom.test.tsx` run for both controlled and uncontrolled modes. They cover blocked selection and `onChange`, preserved value, tab skipping, all options disabled, and deselectable-variant behavior. An `expectGroupChromeDisabled` helper asserts `aria-disabled="true"` and no HTML `disabled` attribute on the group. Snapshot IDs were regenerated.

```tsx
// Before: no effect on the buttons in the deselectable variant
<ToggleGroupControl isDeselectable disabled label="Align" value="left">…</ToggleGroupControl>

// After: every option button is natively disabled; the group has aria-disabled="true"
```

## Contribution

Authored by @mirka, who found the gap while checking whether #57862 could be a quick win. She noticed `disabled` appeared in the Storybook controls though it had never been fully implemented, and the hover style was wrong. Her PR description characterized the prop's earlier addition in #80705 as an accident. @aduth pushed back, noting it was added intentionally (with a link to prior discussion) and asking whether the existing DataViews usage was actually broken, pointing to the faded styling already visible in the Layout Table story. @mirka acknowledged the correction, and the PR was merged with the work reframed as completing the prop's behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

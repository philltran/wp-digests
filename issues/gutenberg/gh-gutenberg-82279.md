# #82279: BorderBoxControl: Make sure the group of controls is associated with the label

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @andrewserong
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Feature] Design Tools`
- **Merged:** [`007d5f9`](https://github.com/WordPress/gutenberg/commit/007d5f98c62df4c9a70827d9fc6de134c2829970)
- **Discussion:** [#82279](https://github.com/WordPress/gutenberg/pull/82279) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`BorderBoxControl` now renders its wrapper as a `role="group"` labelled by its `label` prop via a generated ID and `aria-labelledby`, so assistive tech can associate the overall label with the group of border controls. This brings it in line with `BoxControl`. Consumer-supplied `aria-labelledby` or `aria-label` takes precedence over `label`. When `hideLabelFromVision` is set, the hidden label now renders as a `span` instead of a `label` element.

## Impact

**Plugin & theme developers using `BorderBoxControl`**
- No code changes required. The group gets an accessible name automatically from `label`.
- If you pass `aria-label` or `aria-labelledby`, it now names the group and overrides `label`. Previously these props were spread onto the wrapper `View` without a `group` role.
- If you pass none of `label`, `aria-label`, or `aria-labelledby`, the wrapper gets no `group` role.
- With `hideLabelFromVision`, the hidden label is now a `<span>` rather than a `<label>`. Custom CSS or tests that selected it as a `label` element, or queried it by label role/text association, may need updating.

**Site owners / editors**
- Screen reader users get the border controls (for example in Group block design tools) announced as a named group. No visual change.

**Hosting / headless / REST**
- Not affected.

## Technical details

Changes are in `packages/components/src/border-box-control/border-box-control/component.tsx`:

- `useInstanceId( BorderBoxControl, 'border-box-control-label' )` generates a label ID.
- `labelId` is only set when `label` is present and neither `aria-label` nor `aria-labelledby` is supplied (empty values are treated as absent, matching accessible-name computation). Otherwise the internal label gets no `id`.
- `aria-label` and `aria-labelledby` are now destructured from the `useBorderBoxControl( props )` result and applied explicitly on the wrapper `View`:

```jsx
<View
  className={ className }
  role={ label || ariaLabel || ariaLabelledBy ? 'group' : undefined }
  aria-label={ ariaLabel }
  aria-labelledby={ ariaLabelledBy ?? labelId }
  { ...otherProps }
  ref={ mergedRef }
>
```

- `BorderLabel` accepts an `id`. The visible variant passes it to `BaseControl.VisualLabel`. The hidden variant changes from `<VisuallyHidden as="label">` to `<VisuallyHidden as="span" id={ id }>`.
- The component docblock and `CHANGELOG.md` document the precedence rules.
- New jsdom tests cover: name from label, hidden label, `aria-label` and `aria-labelledby` precedence (including no generated ID on the internal label), a group named only by `aria-label`, fallback when `aria-label=""`, and unique IDs across multiple instances.

Bundle impact is roughly +60 to +91 B on `components/index.min.js`.

## Contribution

Opened as a follow-up to a review comment from @ciampo on #82163, which noted `BorderBoxControl` lacked the label-to-group relationship that `BoxControl` has. @andrewserong authored it, with Claude Code used to generate the change and human review and testing. The merged version went beyond the original description: it handles consumer-supplied ARIA naming props and changes the hidden label to a `span`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

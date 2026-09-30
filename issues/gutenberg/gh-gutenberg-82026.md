# #82026: RadioControl: allow individual options to be disabled

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`8d68310`](https://github.com/WordPress/gutenberg/commit/8d68310a3c95596477f633a6af5ca3869944c7d9)
- **Discussion:** [#82026](https://github.com/WordPress/gutenberg/pull/82026) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`RadioControl` in `@wordpress/components` now accepts an optional `disabled` boolean on each entry in its `options` array. The flag is passed to that option's native radio input, so one choice can be shown as unavailable while the rest of the group stays interactive. Previously the only ways to get this were hiding the option, disabling the whole group, or rebuilding the markup by hand.

## Impact

**Plugin & theme developers (using `@wordpress/components`)**
- New, backwards-compatible, opt-in option field: `options: [ { label, value, disabled?: true } ]`. Existing usage is unaffected.
- Developers who reimplemented `RadioControl` markup and relied on its internal class names to fake per-option disabling can switch to the supported API.
- The `options` type in `types.ts` gains `disabled?: boolean`, so TypeScript consumers get it typed.

**Site owners / hosting / headless:** No action required.

## Technical details

The functional change is one line in `packages/components/src/radio-control/index.tsx`: the per-option `<input type="radio">` now receives `disabled={ option.disabled }`. The group-level `disabled` prop behavior is unchanged.

Other changes in the diff:
- `types.ts`: adds `disabled?: boolean` (`@default false`) to the `options` item type.
- `style.scss`: the pointer-cursor rule changes its selector from `.components-radio-control:not(:disabled) &` to `.components-radio-control__input:not(:disabled) + &`. The cursor is now driven by the sibling input's disabled state rather than the wrapper, so a disabled option no longer shows the control cursor while enabled ones do. Two stray blank lines are also removed. The PR description says the existing disabled indicator styles are reused and the label keeps its normal color and opacity.
- `README.md`: the `options` signature is documented as `{ label, value, description?, disabled? }[]`. This also documents the previously undocumented `description` field.
- Adds a `WithDisabledOption` Storybook story, Jest tests (the group stays enabled, the disabled radio is disabled, clicking it does not fire `onChange`, and enabled options still do), and a CHANGELOG entry under Enhancements.

```jsx
<RadioControl
	label="Visibility"
	selected={ value }
	options={ [
		{ label: 'Public', value: 'public' },
		{ label: 'Private', value: 'private', disabled: true },
	] }
	onChange={ setValue }
/>
```

No new hooks, REST changes, or DB changes.

## Contribution

Opened by @ciampo to close issue #81939 and merged after three bot comments and no human review discussion recorded in the thread. Design feedback on the label styling was taken from a comment on the linked issue. The description states Codex was used to implement the change, run verification, and draft the description.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

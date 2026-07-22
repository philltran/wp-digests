# #79112: BaseControl: add text-wrap: pretty

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jasmussen
- **Labels:** `[Type] Enhancement`, `[Package] Components`
- **Merged:** [`194bb9e`](https://github.com/WordPress/gutenberg/commit/194bb9e0baa6c5d31106fb4c55d5eae462c512ff)
- **Discussion:** [#79112](https://github.com/WordPress/gutenberg/pull/79112) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request applies `text-wrap: pretty` to the `StyledHelp` component inside `BaseControl` to prevent typographic widows in block editor help text. The change improves legibility in the inspector panel without altering component behavior or structure.

## Impact

- **Block & UI developers:** No action required. The CSS enhancement applies automatically to `BaseControl` help text rendered in the block editor.
- **Theme & plugin developers:** No impact unless you are directly overriding `BaseControl` styles or building custom inspector panels that replicate its help text layout.
- **Site owners:** No visible change on the frontend; affects only the block editor UI.

## Technical details

The change modifies `packages/components/src/base-control/styles/base-control-styles.ts` by appending `text-wrap: pretty;` to the `StyledHelp` styled component. This CSS property instructs modern browsers to prefer breaking lines at natural word boundaries rather than leaving single words stranded at the end of a line. The diff adds exactly one line to the template literal and updates the `packages/components/CHANGELOG.md` under `Unreleased > Enhancements`. No JavaScript logic, props, or hooks are altered.

## Contribution

Opened by @jasmussen to address typographic widows observed in the Icon block’s inspector help text. During review, @jameskoster and @juanfra noted that the `Field` component could benefit from the same treatment, but agreed to defer it to a follow-up PR to expedite the merge. The change was merged with co-authors @juanfra and @jameskoster.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

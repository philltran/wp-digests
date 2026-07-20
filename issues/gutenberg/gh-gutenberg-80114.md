# #80114: Editor: restore the behavior of only showing back button focus ring on :focus-visible

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Bug`, `[Package] Edit Site`
- **Merged:** [`fbb3525`](https://github.com/WordPress/gutenberg/commit/fbb352583d5092676b4e84c6d1ea3a8554cb59d0)
- **Discussion:** [#80114](https://github.com/WordPress/gutenberg/pull/80114) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request restores the intended focus behavior for the Site Editor sidebar back button, ensuring the focus ring only appears when triggered by keyboard navigation (`:focus-visible`). The behavior was accidentally removed in a prior change, causing the button to display a focus ring on mouse clicks. The fix re-applies the `:focus-visible` pattern using a simplified CSS selector rather than reverting the previous commit.

## Impact

- **Site editors & theme/plugin developers:** No code changes required. The Site Editor UI now correctly hides the focus ring when navigating with a mouse, improving accessibility compliance and visual consistency.
- **Hosting & platform teams:** No configuration or migration steps needed. The change is contained within the Gutenberg build output.
- **No action required** for existing sites or custom themes/plugins unless they directly override the `sidebar-button` component styles.

## Technical details

The change modifies `packages/edit-site/src/components/sidebar-button/style.scss`. It introduces a new rule that explicitly removes the outline for mouse-triggered focus events:

```scss
&:focus:not(:focus-visible) {
    outline: none;
}
```

This rule is placed before the existing `&:hover:not(:disabled,[aria-disabled="true"])`, `&:focus-visible`, and `&:focus` selectors. By explicitly clearing the outline on `:focus` when `:focus-visible` is not active, the component correctly suppresses the focus ring for pointer interactions while preserving it for keyboard navigation. The compiled CSS output increases by approximately 39 bytes across the `edit-site` style files.

## Contribution

Opened and merged by @fushar, with co-authorship from @t-hamano. The PR addresses a regression introduced in #80029. Rather than performing a full revert, the author proposed a simplified CSS selector to achieve the same `:focus-visible` behavior. The change was reviewed, received thumbs-up, and merged without significant design debate.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# #78901: Autocomplete: Add Group and GroupLabel primitives

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `Needs Design Feedback`, `[Package] UI`
- **Merged:** [`23b8035`](https://github.com/WordPress/gutenberg/commit/23b80358e717f86ba3cba1c3dfe0021117bd98a3)
- **Discussion:** [#78901](https://github.com/WordPress/gutenberg/pull/78901) · 11 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `@wordpress/ui` package now exports `Autocomplete.Group` and `Autocomplete.GroupLabel` primitives, enabling developers to organize autocomplete suggestions into labeled sections. This addition supports the ongoing refactoring of the command palette away from the `cmdk` library and provides a native grouping capability for the `Autocomplete` component.

## Impact

- **Plugin & theme developers:** Can now structure autocomplete dropdowns into categorized sections using the new subcomponents. No breaking changes; existing `Autocomplete` usage remains compatible.
- **Command palette consumers:** The command palette will eventually leverage these primitives to resolve upstream `cmdk` accessibility issues, though the migration is not yet complete.
- **No action required** for sites not using the `@wordpress/ui` autocomplete component.

## Technical details

The diff introduces `packages/ui/src/form/primitives/autocomplete/group.tsx` and `group-label.tsx`, which wrap `@base-ui/react/autocomplete`'s `Group` and `GroupLabel` elements. Both components forward refs and accept standard React props via `AutocompleteGroupProps` and `AutocompleteGroupLabelProps` (defined in `types.ts`). Styling is applied in `packages/ui/src/utils/css/item-popup.module.css`, adding `.group` margin spacing and `.group-label` flex layout with neutral-weak foreground color. The exports are registered in `index.ts`. 

Additionally, the diff replaces hardcoded item heights with design system tokens:
```diff
- --wp-ui-popup-item-height: 40px;
+ --wp-ui-popup-item-height: var(--wpds-dimension-size-lg);
```
Storybook fixtures and unit tests for ref forwarding are included.

## Contribution

Opened by @t-hamano and merged after design feedback from @jasmussen, @fcoveram, and @jameskoster. Reviewers debated the default all-caps styling for group labels, noting alignment with the existing `Menu` component and broader design system consistency, but agreed to ship the current approach as a baseline that can be overridden. The PR was co-authored by six additional contributors during review.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

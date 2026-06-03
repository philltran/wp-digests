# #78399: Add Combobox primitives

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`c0abcb3`](https://github.com/WordPress/gutenberg/commit/c0abcb352ee6d8dd45acefc691fbe9eb61b40f68)
- **Discussion:** [#78399](https://github.com/WordPress/gutenberg/pull/78399) · 4 comments · 1 reactions

## Summary

A new `Combobox` primitive set has been added to `@wordpress/ui` (`packages/ui`), providing low-level, composable building blocks for searchable selection UIs. The set covers the full combobox anatomy — trigger, input, popup, list, item, chip-with-remove, clear, empty, and portal — built as thin wrappers around `@base-ui/react/combobox`. This gives Gutenberg and downstream consumers a shared, accessible foundation for both single-select and chip-based multi-select patterns without having to build keyboard management and ARIA semantics from scratch.

## Impact

**Plugin & theme developers (block editor extensions)**
- New opt-in API only; no existing code is affected.
- `Combobox` is exported from `@wordpress/ui`, not `@wordpress/components`. Consumers must already depend on `@wordpress/ui`, which is a separate, newer package.
- The primitives are unstyled at the primitive level; consuming code is responsible for visual treatment.
- No deprecations, no removed APIs. No action required if you are not building a combobox UI.

**Platform / hosting**
- No server-side changes, no new REST endpoints, no DB schema changes.
- Bundle size impact reported as 0 B in the tracked build artifacts (the `@wordpress/ui` module is not yet included in the measured script bundles).

## Technical details

All new files land under `packages/ui/src/form/primitives/combobox/`. The implementation wraps `@base-ui/react/combobox` — confirmed by the import in the partial diff:

```tsx
import { Combobox as _Combobox } from '@base-ui/react/combobox';
```

Each sub-component (`chip-with-remove.tsx` is the only file visible in the diff excerpt) uses `forwardRef` from `@wordpress/element`, `clsx` for class name composition, `__` from `@wordpress/i18n` for translatable strings, and icon assets from `@wordpress/icons`. The CHANGELOG entry in `packages/ui/CHANGELOG.md` records the addition under "Unreleased".

The nine named sub-components described in the PR are:
- **Trigger** — button that opens the popup
- **Input** — the text search field
- **Popup** — floating container (uses portal by default)
- **Portal** — explicit portal escape hatch
- **List** — scrollable option list
- **Item** — individual option row
- **Chip** (chip-with-remove) — token representing a selected value with a dismiss button
- **Clear** — clears all selections
- **Empty** — empty-state slot

The diff shows no changes to `packages/components`, block.json files, REST schema, or WordPress core PHP. This is a purely JavaScript/TypeScript addition to the `@wordpress/ui` package.

## Contribution

The PR was opened and merged by @mirka. @ciampo (Marco Ciampini) reviewed the work; the `props-bot` comment lists both as co-authors. @mirka noted in a comment that all review feedback had been addressed before merge. The discussion thread was brief (4 comments), with no record of alternative design approaches being debated in the PR itself. The commit hash is `c0abcb3`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

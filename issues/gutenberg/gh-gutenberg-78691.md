# #78691: Tooltip migration: fields + media-editor + media-fields + global-styles-ui (4/5)

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Fields`
- **Merged:** [`ac7cf83`](https://github.com/WordPress/gutenberg/commit/ac7cf834bb5cf1d7d7125470b97a28d55a555c32)
- **Discussion:** [#78691](https://github.com/WordPress/gutenberg/pull/78691) · 3 comments · 0 reactions

## Summary

Part 4 of a 5-PR series migrates five `Tooltip` call sites across `@wordpress/fields`, `@wordpress/media-editor`, `@wordpress/media-fields`, and `@wordpress/global-styles-ui` from the legacy `Tooltip` exported by `@wordpress/components` to the new compositional `Tooltip` from `@wordpress/ui` (backed by base-ui). The goal is to consolidate all tooltip behavior — delays, providers, positioner — in a single implementation. `@wordpress/ui` is now a runtime dependency of `@wordpress/media-fields` and `@wordpress/global-styles-ui` as a result. The `@wordpress/ui` `Tooltip` will be declared the ESLint-recommended component once part 5 lands.

## Impact

**Plugin and theme developers**
- No breaking changes. The `Tooltip` export from `@wordpress/components` is not deprecated in this PR and remains available.
- Code importing `Tooltip` from `@wordpress/components` continues to work without modification.

**Package maintainers / Gutenberg contributors**
- `@wordpress/ui` is now a declared runtime dependency of `@wordpress/media-fields` and `@wordpress/global-styles-ui`. Projects that vendor or tree-shake these packages will pull in the base-ui-backed tooltip implementation.
- Each migrated import is paired with a temporary `// eslint-disable-next-line @wordpress/use-recommended-components` directive; the recommendation enforcement flip is deferred until after part 5 merges.
- The `Tooltip` inside `@wordpress/components` itself (`Button`'s internal tooltip, `TooltipInternalContext`) is explicitly out of scope and still uses the legacy path.

**No action required** for anyone consuming these packages through the standard WordPress script dependencies.

## Technical details

Each of the five call sites changes its import source:

```js
// Before
import { Tooltip } from '@wordpress/components';

// After
// eslint-disable-next-line @wordpress/use-recommended-components
import { Tooltip } from '@wordpress/ui';
```

The five files touched:
- `packages/fields/src/components/media-edit` — icon-only media picker trigger.
- `packages/fields/src/fields/pattern-title` — "This pattern cannot be edited." tooltip on theme pattern rows.
- `packages/media-editor/src/components/media-form/sidebar-datetime-view` — full datetime on hover in the media library sidebar.
- `packages/media-fields/src/filename/view` — full filename on hover in the media DataView for truncated names.
- `packages/global-styles-ui/src/variations/variation` — variation title tooltip in the Global Styles panel.

`@wordpress/ui` is added to the `dependencies` field in the `package.json` of `@wordpress/media-fields` and `@wordpress/global-styles-ui` (it was already present in the other affected packages from earlier batches).

Hover-only triggers remain intentionally hover-only (no `focus` trigger) at each site; the rationale is documented inline in the source. The new `@wordpress/ui` `Tooltip` is compositional and built on base-ui, centralizing delay configuration and requiring a `Tooltip.Provider` higher in the tree — that provider will be wired at the shell level in part 5.

Build size impact is negligible: +94 B compressed across all bundles, with the bulk in `build/scripts/editor/index.min.js` (+75 B) and `build/scripts/media-utils/index.min.js` (+20 B).

## Contribution

Opened and merged by @ciampo as the fourth installment of a planned five-PR series (preceding batches: #78411 for `block-editor`/`block-directory` plus the original codemod; #78466 for `editor`/`edit-post`/`edit-site`; #78470 for `dataviews`). The mechanical rewrite was performed by the codemod introduced in #78411; import ordering and per-line ESLint disable directives were applied by hand. The PR description notes Cursor AI assistance with author review and verification. A parallel codemod cleanup PR (#78669) and the final part-5 PR (which adds the shell-level `Tooltip.Provider` and flips the ESLint recommendation) are the outstanding follow-ups.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

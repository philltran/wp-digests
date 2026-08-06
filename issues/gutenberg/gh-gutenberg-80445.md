# #80445: ToolsPanel: Migrate styles to an SCSS Module

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Components`, `Backported to WP Core`
- **Merged:** [`6749a0b`](https://github.com/WordPress/gutenberg/commit/6749a0bcd5842b48e9f2b880f08457f2443b90f4)
- **Discussion:** [#80445](https://github.com/WordPress/gutenberg/pull/80445) · 10 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `ToolsPanel` component has been migrated from Emotion-based CSS-in-JS to a compiled SCSS module (`style.module.scss`). This change fixes a visual regression where the panel’s heading typography was incorrectly inheriting styles from the `Heading` component due to an insertion-order conflict exposed by the `View` migration. By moving styles to a static module, the cascade is resolved at the source without relying on Emotion’s dynamic class composition.

## Impact

- **Plugin & theme developers:** No direct API changes. The `ToolsPanel` component’s public props, DOM structure, and legacy class names (e.g., `components-tools-panel`) remain unchanged.
- **Block developers:** If you are targeting `ToolsPanel` via CSS in your block editor stylesheets, you may now target the generated SCSS module class names (e.g., `tools-panel-heading`) alongside the legacy `components-*` classes.
- **Platform & hosting:** No action required. The change is internal to the `@wordpress/components` package and ships with WordPress 6.7+ (and backported to 7.1).
- **Breaking changes:** None. Emotion is no longer used for this component’s styling, but the external interface and specificity rules are preserved.

## Technical details

The diff removes `packages/components/src/tools-panel/styles.ts` (which relied on `@emotion/styled`, `css`, and `cx`) and introduces `packages/components/src/tools-panel/style.module.scss`. All styling logic is now handled via `clsx` in the component hooks (`useToolsPanelHeader`, `useToolsPanelItem`). The SCSS module defines classes like `.tools-panel`, `.tools-panel-header`, `.tools-panel-heading`, and `.tools-panel-item`, with specificity carefully matched to the previous Emotion `&&&&` and `&&` patterns to override `DropdownMenu` and `BaseControl` defaults. The `View` component now receives static class names instead of Emotion-generated fragments, eliminating the stylesheet insertion-order conflict that broke heading typography.

**Before (hook pattern):**
```ts
const cx = useCx();
const headingClassName = useMemo(() => cx(styles.ToolsPanelHeading), [cx]);
```

**After (hook pattern):**
```ts
import clsx from 'clsx';
import styles from '../style.module.scss';
// ...
headingClassName: styles['tools-panel-heading'],
className: clsx(styles['tools-panel-header'], className),
```

## Contribution

Opened as a follow-up to the `View` migration, the PR was reviewed and merged with co-authors @mirka, @juanfra, and @t-hamano. During review, @t-hamano identified that this fix also resolves a related regression in #80729 and requested a backport to the `wp/7.1` branch. The cherry-pick encountered a merge conflict that required manual resolution before the backport PR was created. The author noted that Codex was used to audit the complete styling surface and validate the generated CSS.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

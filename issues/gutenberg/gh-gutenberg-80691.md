# #80691: Components: Remove private Theme component

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Breaking Change`, `[Package] Components`
- **Merged:** [`b2ab2cc`](https://github.com/WordPress/gutenberg/commit/b2ab2ccba568a4031493cb7d087b2a2d07e0dc70)
- **Discussion:** [#80691](https://github.com/WordPress/gutenberg/pull/80691) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The private `Theme` component has been removed from `@wordpress/components`. This component was an internal experiment for dynamically setting `--wp-components-color-*` CSS custom properties at runtime. With the canvas loader no longer relying on it, the component is now dead code, and `@wordpress/theme` is the designated replacement for runtime theming.

## Impact

- **Plugin & theme developers:** If you were importing or using the private `Theme` component from `@wordpress/components`, your code will break. The component is no longer exported.
- **Action required:** Replace usage with `@wordpress/theme` or manage CSS custom properties directly. No migration script is provided.
- **Public API consumers:** No action required. This change only affects developers using private or experimental APIs.

## Technical details

The diff removes the entire `packages/components/src/theme/` directory, including `index.tsx`, `color-algorithms.ts`, `style.module.scss`, and associated tests/stories. It also strips `Theme` from the private API registry in `packages/components/src/private-apis.ts` and removes it from `docs/private-apis.md`. The component previously accepted `accent` and `background` props, used `colord` to generate derived color variables, and injected them via a `style` object on a wrapper element.

**Before (registry export):**
```diff
-import Theme from './theme';
...
-	Theme,
```

Component styles still reference `--wp-components-color-*` tokens, which will be gradually migrated to wpds tokens.

## Contribution

Opened and merged by @mirka, with co-authorship from @manzoorwanijk. The change was triggered by the canvas loader migration in #80688, which eliminated the component's only runtime dependency. During review, @aduth questioned the semantic versioning tension around labeling private API removals as breaking changes. @mirka acknowledged the inconsistency but noted that some private APIs remain in active use, making blanket exemptions impractical.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

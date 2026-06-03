# #56880: Changed labels to consistently use Patterns in favor of Block patterns

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @colinduwe
- **Labels:** `[Type] Enhancement`, `[Package] Block editor`, `First-time Contributor`
- **Merged:** [`e797bcc`](https://github.com/WordPress/gutenberg/commit/e797bcc6988baba46d7491c2fc87fdaf6f7d3504)
- **Discussion:** [#56880](https://github.com/WordPress/gutenberg/pull/56880) · 3 comments · 0 reactions

## Summary

The Block editor UI labels have been updated from "Block patterns" to simply "Patterns" across the inserter, success notices, and onboarding screens. This change standardizes terminology to align with core documentation and reduce redundancy, resolving issue #49617. No functional behavior has changed; only user-facing strings within the Gutenberg package were updated.

## Impact

- **Plugin & theme developers**: No breaking changes or API alterations. The `BlockPatternsList` component's default `label` prop was changed to `'Patterns'`, but custom implementations explicitly passing a label will continue to work as before.
- **Editors / Site owners**: UI text in the pattern inserter and insertion success notices now reads "Patterns" instead of "Block patterns". No migration or configuration required.
- **Automated testing / e2e scripts**: Screen selectors referencing `name="Block patterns"` or listbox roles will need updates to match the new `name="Patterns"` locator in Playwright/test suites.

## Technical details

The diff modifies default string literals and test locators across the `@wordpress/block-editor` package:
- `packages/block-editor/src/components/block-patterns-list/index.js`: Default `label` prop for `BlockPatternsList` changed from `'Block patterns'` to `'Patterns'`.
- `packages/block-editor/src/components/inserter/hooks/use-patterns-state.js`: Success notice sprintf updated to `'Pattern "%s" inserted.'`.
- `packages/block-editor/src/components/inserter/search-results.js`: Visually hidden title updated to match the new label.
- Test suites (`allowed-patterns.spec.js`, `inserting-blocks.spec.js`, `patterns.spec.js`, `template-part.spec.js`): Playwright/screen query locators updated from `name="Block patterns"` to `name="Patterns"`.
- `widgets/welcome/render.tsx`: Onboarding banner description text updated.
No PHP hooks, REST endpoints, or block schema changes were introduced; this is strictly a frontend string localization update within the JS/TS package.

## Contribution

Opened by @colinduwe to address terminology consistency raised in #49617. The PR was reviewed, rebased onto trunk by @talldan, and merged with co-authors @masteradhoc, @richtabor, @jordesign, and @afercia. The change was treated as a straightforward string adjustment without architectural debate, focusing solely on UI label unification across the editor and welcome widgets.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

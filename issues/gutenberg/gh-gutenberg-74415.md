# #74415: UI: add `Button`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] UI`
- **Merged:** [`253bb2a`](https://github.com/WordPress/gutenberg/commit/253bb2af362b41e4956399cf888332c624291487)
- **Discussion:** [#74415](https://github.com/WordPress/gutenberg/pull/74415) · 6 comments · 0 reactions

## Summary

The `@wordpress/ui` package adds a new `Button` component designed to modernize interactive UI patterns previously handled in `@wordpress/components`. It leverages CSS variables from `@wordpress/theme` for theming, supports explicit tone/variant/size props, and natively manages accessibility states using standard HTML attributes like `aria-pressed`. The component also introduces a built-in loading state while removing an internal tooltip dependency to reduce bundle size.

## Impact

['- **Block & UI Developers**: Gain access to a primitive `Button` export from `@wordpress/ui` with explicit design token support and cleaner variant prop patterns.', '- **Plugin & Theme React Developers**: No breaking changes. The component serves as an optional modern replacement; existing `@wordpress/components` patterns remain fully functional.', '- **Hosting & Platform Teams**: Zero configuration impact. Package dependencies are resolved transparently through standard npm/yarn workflows.', '- **No action required** unless explicitly adopting the new UI primitives in client-side or editor-facing React code.']

## Technical details

['- Adds the `Button` implementation to the `packages/ui` package, establishing it as a layout-independent primitive alongside recent additions like `Icon` and `Field`.', '- Updates `packages/ui/package.json` to declare `@ariakit/react@^0.4.15` and `@wordpress/a11y` as direct dependencies, satisfying accessibility enforcement and state management requirements.', '- Modifies `.eslintrc.js` to permit restricted imports (`@ariakit`, `framer-motion`) specifically for `packages/ui/src/**`, aligning linting rules with the new primitive package boundary.', '- **API shift**: Replaces custom React props for interactive states with standard HTML attributes (e.g., `aria-pressed`). Disabled states are inherently accessible by default, and a `loading` prop is exposed without carrying forward legacy tooltip logic.', '```jsx\n// Documented API pattern via @wordpress/ui\n<Button tone="primary" size="small" aria-pressed={isActive} loading={isSaving}>\n  Toggle\n</Button>\n```']

## Contribution

['Open and led by @ciampo as part of a broader UI refactoring series (#71196), with technical review and co-authorship from @aduth, @mirka, and @jameskoster.', 'During the review round, the team considered bundling destructive variants inline to satisfy legacy editor use cases, but agreed to defer this scope to a follow-up PR to keep the initial merge tightly scoped.', 'All feedback was addressed in iterative commits, with CI passing prior to the final merge commit.']

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

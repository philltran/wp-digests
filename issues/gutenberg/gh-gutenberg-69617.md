# #69617: Enhance sandbox component to accept sandbox prop

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Imran92
- **Labels:** `[Type] Enhancement`, `[Feature] UI Components`, `[Package] Components`
- **Merged:** [`839f5c6`](https://github.com/WordPress/gutenberg/commit/839f5c67e0d2a869611c6f5d3f1a9201ffc34ab1)
- **Discussion:** [#69617](https://github.com/WordPress/gutenberg/pull/69617) · 10 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `SandBox` component in `@wordpress/components` now accepts an optional `allowPopups` boolean prop. When enabled, it appends `allow-popups` to the iframe's `sandbox` attribute, allowing links and `window.open` calls inside sandboxed content to navigate. This resolves a limitation where sandboxed previews previously blocked all outbound navigation and enables consumers like Jetpack VideoPress to render interactive embeds safely.

## Impact

- **Plugin & theme developers:** No action required for existing code. Developers embedding interactive content in `SandBox` can now pass `allowPopups={ true }` to enable link navigation.
- **Hosting & platform teams:** No changes needed.
- **Headless & REST consumers:** Unaffected.
- **Security note:** The default remains strictly sandboxed. Opting into popups does not bypass the existing `allowSameOrigin` security boundary or alter the component's default behavior.

## Technical details

The diff modifies `packages/components/src/sandbox/index.tsx` and `packages/components/src/sandbox/types.ts`. Both `IsolatedSandBox` and `SameOriginSandBox` now accept `allowPopups?: boolean` (defaulting to `false`). Instead of hardcoding the `sandbox` attribute string, the component uses `clsx` to dynamically construct it:

```tsx
// Before
sandbox="allow-scripts allow-presentation"

// After
const sandbox = clsx( 'allow-scripts', 'allow-presentation', {
  'allow-popups': allowPopups,
} );
sandbox={ sandbox }
```

The `SandBoxContentProps` interface in `types.ts` is updated to include the new prop with JSDoc documentation. Unit tests in `test/index.tsx` verify that the attribute is absent by default and correctly appended when `allowPopups` is passed. No hooks, filters, or REST schema changes are involved.

## Contribution

Opened to unblock interactive embed fixes in Jetpack, the PR underwent review focused on API surface design. A reviewer questioned whether a generic `sandbox` string override would be more future-proof given the 14+ `allow-*` iframe tokens. The team evaluated merging arbitrary sandbox tokens but rejected it due to parsing complexity and the risk of accidentally overriding default security flags. The group settled on the targeted `allowPopups` boolean to preserve the existing `allowSameOrigin`-driven boolean-flag architecture and maintain strict security defaults.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

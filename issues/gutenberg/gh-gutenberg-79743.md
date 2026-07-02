# #79743: ExternalLink: Stop setting default rel

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Components`, `[Package] Editor`
- **Merged:** [`29cafb8`](https://github.com/WordPress/gutenberg/commit/29cafb855837ecfa90b5a2d716094d73a94ec253)
- **Discussion:** [#79743](https://github.com/WordPress/gutenberg/pull/79743) · 7 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The `ExternalLink` component in `@wordpress/components` no longer automatically sets the `rel="external noopener"` attribute on rendered anchor tags. This aligns the component with modern browser behavior where `target="_blank"` implicitly prevents reverse-tabnabbing, and shifts responsibility for `rel` serialization entirely to the consumer.

## Impact

- **Plugin & theme developers:** If you use `wp.components.ExternalLink` or wrap it in your own components, you must now explicitly pass a `rel` prop (e.g., `rel="noopener noreferrer"`) if that security/a11y behavior is required.
- **Block & editor authors:** Any block UI relying on the post-publish panel or other `ExternalLink` instances will no longer receive auto-generated `rel` attributes in the rendered DOM. No visual or accessibility regression is introduced.
- **Site owners / General consumers:** No action required unless you directly interact with editor components via custom scripts or automated testing that asserts specific anchor attributes.

## Technical details

The change modifies `packages/components/src/external-link/index.tsx`. Previously, `UnforwardedExternalLink` destructured a `rel` prop and constructed an `optimizedRel` array that forced `'external'` and `'noopener'` tokens:
```
diff - const { href, children, className, rel = '', ...additionalProps } = props;
- const optimizedRel = [ ...new Set( [ ...rel.split( ' ' ), 'external', 'noopener' ].filter( Boolean ) ), ].join( ' ' );
+ const { href, children, className, ...additionalProps } = props;
```
The `rel={ optimizedRel }` attribute was also removed from the JSX template. Consequently, the component now leaves `rel` unset by default and passes through any consumer-provided value unchanged.

Test expectations were updated in `packages/editor/src/components/post-publish-panel/test/__snapshots__/index.js.snap`, removing `rel="external noopener"` from two post-publish panel anchor assertions. The `@wordpress/components/CHANGELOG.md` explicitly marks this as a breaking change.

## Contribution

Opened by @ciampo and merged as commit `29cafb8`. The PR originated from @simison's review of a broader initiative to stop injecting `noreferrer`/`noopener` by default (tracked in #26914). @aduth confirmed modern browsers implicitly apply `noopener` behavior for `target="_blank"`, prompting @ciampo to pivot the PR from research to implementation. The change was merged after verifying parity with the `@wordpress/ui` `Link` component and confirming zero keyboard accessibility impacts.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

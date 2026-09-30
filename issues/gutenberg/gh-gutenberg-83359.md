# #83359: Theme: Preserve global token references in Lightning CSS

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] Theme`
- **Merged:** [`265f20f`](https://github.com/WordPress/gutenberg/commit/265f20fda265f6a5137e13b5a2700d2b8fd1d2eb)
- **Discussion:** [#83359](https://github.com/WordPress/gutenberg/pull/83359) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `@wordpress/theme` Lightning CSS plugin (`lightningcss-ds-token-fallbacks.mjs`) previously rebuilt `var()` expressions from the token name alone when injecting design-token fallbacks, discarding CSS Modules `from global` reference metadata. With `cssModules.dashedIdents` enabled, this caused global custom properties to be incorrectly hashed. The fix pre-parses all token fallbacks through Lightning CSS at module load time and combines them with the original variable's `name` node (preserving `from`), so `var(--wpds-dimension-gap-sm from global)` now correctly retains its global reference and its nested fallback variables (e.g. `--wp-admin-*`) also remain unscoped.

## Impact

- **Theme developers using `@wordpress/theme` build plugins with Lightning CSS + CSS Modules (`dashedIdents: true`):** Previously, `from global` references on design tokens were silently dropped and the token name was hashed as if it were local. After this fix, the reference is preserved and nested fallback variables stay global so admin overrides still apply. No code changes needed—this is a build-pipeline fix.
- **Plugin & theme developers using PostCSS or Stylelint token-fallback plugins:** No behavior change. The PostCSS and Stylelint paths are untouched.
- **No new dependencies, no breaking API changes, no configuration changes required.**

## Technical details

The core change is in `packages/theme/lightningcss-plugins/lightningcss-ds-token-fallbacks.mjs`. Previously the visitor called `addFallbackToVar(\`var(${tokenName})\`)`, a regex-based string replacement that produced a raw `var()` string with no reference metadata. The new implementation:

1. **Pre-parses fallbacks at module load.** A one-time `transform()` call from `lightningcss` parses a synthetic `:root` block containing every token as `var(<token>, <fallback>)`. A `Variable` visitor captures each parsed fallback into a module-level `Map<string, TokenOrValue[]>` called `parsedFallbacks`. During this parse, any `from: null` (i.e. `from global`) in the fallback is rewritten to `{ type: 'global' }` via a `JSON.parse(JSON.stringify(...))` replacer so CSS Modules won't scope nested variables.

2. **Preserves the original `name` node.** In the main visitor, the returned `var` node uses `variable.name` (which carries `from` metadata) when present, falling back to `{ ident: variable.name.ident }` otherwise. The fallback is a `structuredClone` of the cached parsed fallback, so a composed visitor mutating one build's fallback cannot corrupt a later build.

3. **Explicit error for missing parsed fallbacks.** If a token exists in `tokenFallbacks` but has no entry in `parsedFallbacks`, the visitor throws `No parsed fallback for design token: <name>`. The existing unknown-token error (from `getTokenFallback`) is preserved for tokens not in the map at all.

A new `getTokenFallback(tokenName, tokenFallbacks, { escapeQuotes })` function is extracted from `add-fallback-to-var.mjs` and re-exported through `ds-token-fallbacks.mjs`, so the Lightning CSS plugin can call it without duplicating the unknown-token check.

```js
// Before (simplified): the visitor returned a raw string
return { raw: addFallbackToVar(`var(${tokenName})`) };

// After: structured node preserving `from` metadata
return {
  type: 'var',
  value: {
    name: variable.name.from ? variable.name : { ident: variable.name.ident },
    fallback: structuredClone(fallback),
  },
};
```

Tests in `build-plugin-parity.test.ts` verify: (a) four token/fallback combinations retain `from global` and produce the expected output with `dashedIdents: true`; (b) a composed visitor that mutates a fallback's length value does not affect a subsequent independent transform; (c) a token present in `tokenFallbacks` but absent from the parsed cache throws the explicit error.

## Contribution

Opened by @ciampo as a follow-up to #80401. The original PR included a broader parser expansion (shared `postcss-value-parser`-based token-reference detection across PostCSS, Stylelint, and Lightning CSS) that had been proposed in #82169 and #82209. @mirka questioned whether the parser expansion warranted reopening a previously closed decision, and suggested separating the miscellaneous Lightning CSS fixes. @ciampo removed the parser expansion, leaving only the `from global` fix. @jsnajdr weighed in on the broader question of AI-generated defensive code and the value of human judgment in rejecting over-engineering. @ciampo agreed to open a separate PR for the parser approach. The merged commit includes only the Lightning CSS fallback-preservation fix; the parser expansion was deferred for separate discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

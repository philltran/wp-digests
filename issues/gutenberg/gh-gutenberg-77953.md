# #77953: Blocks: Migrate Markdown converter from showdown to marked

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Blocks`, `Needs Dev Note`
- **Merged:** [`28d414f`](https://github.com/WordPress/gutenberg/commit/28d414f1327652e2b49e784ddc12098768991c62)
- **Discussion:** [#77953](https://github.com/WordPress/gutenberg/pull/77953) · 8 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Replaces the legacy `showdown` Markdown parser with `marked` in the `@wordpress/blocks` package to handle Markdown paste handling in the editor. This migration reduces the client-side bundle size, resolves a known ReDoS vulnerability (`CVE-2024-1899`) in the unmaintained `showdown` dependency, and aligns parsing with modern CommonMark/GFM standards. The change is transparent to end users but improves performance and security for editor operations.

## Impact

- **Plugin & theme developers:** No direct API changes. The `markdownConverter` function in `@wordpress/blocks` remains the public entry point, but its internal implementation and output formatting (e.g., trailing newlines in code blocks) may shift slightly. Developers relying on exact HTML output from `markdownConverter` should verify edge cases.
- **Hosting & platform:** Resolves `CVE-2024-1899` in the `showdown` dependency, clearing Dependabot alerts for sites using the Gutenberg plugin or WordPress 6.6+.
- **Site owners:** No action required; the editor's paste behavior remains functionally identical.

## Technical details

The diff swaps `showdown@1.9.1` for `marked@18.0.3` in `packages/blocks/package.json` and updates `packages/blocks/src/api/raw-handling/markdown-converter.ts`. The legacy `showdown.Converter` call is replaced with a `marked.Marked` instance configured with `gfm: true` and `breaks: true`. To preserve backward compatibility, a custom renderer overrides `code` and `text` tokens: the `code` renderer strips the trailing newline that `marked` appends by default, and the `text` renderer applies a custom `escapeBodyText` function that skips escaping quotes (`"` and `'`) so shortcodes like `[gallery ids="123"]` survive for downstream shortcode processing. The public export now calls `converter.parse( text, { async: false } )` instead of `converter.makeHtml()`.

**Before:**
```ts
const converter = new showdown.Converter( { noHeaderId: true, tables: true } );
return converter.makeHtml( text );
```

**After:**
```ts
const converter = new Marked( { gfm: true, breaks: true, renderer: { code, text } } );
return converter.parse( text, { async: false } );
```

Test expectations in `packages/blocks/src/api/raw-handling/test/markdown-converter.js` were updated to reflect minor whitespace differences, and `test/unit/jest.config.js` was patched to include `marked` in `transformIgnorePatterns`.

## Contribution

Opened by @Mamaduka as a follow-up to an earlier attempt, the PR was merged after review cycles that addressed testing instructions and output alignment. @ahuseyn and @wndyy highlighted the security benefit of removing `showdown` due to `CVE-2024-1899`, which helped accelerate the merge. The author noted that a developer note would be drafted for the next major release, and the change shipped in the merged commit.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

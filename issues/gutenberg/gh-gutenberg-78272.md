# #78272: Bump fast-xml-builder from 1.0.0 to 1.2.0

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dependabot[bot]
- **Labels:** `[Type] Build Tooling`, `javascript`, `dependencies`
- **Merged:** [`d9777e3`](https://github.com/WordPress/gutenberg/commit/d9777e332401736ee0e04af6a861ebdd5122e925)
- **Discussion:** [#78272](https://github.com/WordPress/gutenberg/pull/78272) · 10 comments · 0 reactions

## Summary

`fast-xml-builder` is bumped from 1.0.0 to 1.2.0 in Gutenberg's build toolchain. The upgrade spans twelve patch/minor releases and carries several security fixes alongside new features. Most notably, versions 1.1.5–1.1.7 patched injection vulnerabilities in XML comment, CDATA, and attribute-value serialisation. The bump is a build-time dependency update only; it has no effect on published WordPress/Gutenberg packages or runtime behaviour.

## Impact

**Plugin & theme developers / site owners / hosting & platform / headless & REST consumers**
- No action required. `fast-xml-builder` is a build-time dependency used internally by Gutenberg's toolchain; it is not shipped in any npm package or WordPress core release consumed by end users.

**Gutenberg contributors / local build environments**
- Running `npm install` after pulling this change will pull in two new transitive dependencies: `path-expression-matcher@^1.5.0` and `xml-naming@^0.1.0` (both MIT-licensed).

## Technical details

The change is confined to `package-lock.json`. The lock file entry for `node_modules/fast-xml-builder` moves from `1.0.0` to `1.2.0` and gains a `dependencies` block that was absent before:

```json
// Before (1.0.0)
"node_modules/fast-xml-builder": {
  "version": "1.0.0",
  "license": "MIT"
}

// After (1.2.0)
"node_modules/fast-xml-builder": {
  "version": "1.2.0",
  "license": "MIT",
  "dependencies": {
    "path-expression-matcher": "^1.5.0",
    "xml-naming": "^0.1.0"
  }
}
```

Two new top-level `node_modules` entries are added to the lock file for those transitive dependencies (`path-expression-matcher@1.5.0`, `xml-naming@0.1.0`). No `package.json` changes are present in the diff, confirming this is a lockfile-only update. Security-relevant changelog entries from the intermediate releases include: comment/CDATA injection fixes (1.1.5–1.1.6) and an attribute-value quote-escape fix (1.1.7). New capabilities added — `sanitizeName` option and `xml-naming` integration for tag/attribute name validation (1.2.0) — are unused by Gutenberg at this time but available to the library's callers.

## Contribution

Opened automatically by `@dependabot[bot]`. `@desrosj` (Jonathan Desrosiers) shepherded the PR through repeated rebase requests after Dependabot encountered a configuration-deletion issue that prevented automatic rebasing; he ultimately used `@dependabot recreate` to regenerate the PR before merging it at commit `d9777e3`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

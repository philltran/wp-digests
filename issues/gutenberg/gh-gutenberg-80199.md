# #80199: Add Table of Contents e2e user story coverage

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @getdave
- **Labels:** `[Type] Enhancement`, `[Block] Table of contents (experimental)`
- **Merged:** [`63fc973`](https://github.com/WordPress/gutenberg/commit/63fc97340b50ff793e8411887e82f6a3cb1c4748)
- **Discussion:** [#80199](https://github.com/WordPress/gutenberg/pull/80199) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds comprehensive Playwright e2e test coverage for the experimental Table of Contents block. The tests validate editor behavior, frontend navigation, heading level filtering, list styling, and custom heading-source blocks, while marking desired but unimplemented behaviors (like dynamic frontend rendering) as `test.fixme`. This establishes an automated baseline ahead of the block’s potential stabilization.

## Impact

- **Gutenberg contributors & block maintainers:** New automated coverage reduces regression risk during future ToC block development and stabilization.
- **Plugin & theme developers:** No breaking changes or API shifts. The experimental block’s behavior remains unchanged; developers relying on it can now test against a stable e2e baseline.
- **Site owners & platform teams:** No action required. The change is test-only and does not affect runtime behavior or configuration.

## Technical details

Introduces `test/e2e/specs/editor/blocks/table-of-contents.spec.js` and a test-only plugin (`packages/e2e-tests/plugins/table-of-contents-heading-source.php`) that registers `e2e-tests/table-of-contents-heading-source` and `e2e-tests/table-of-contents-heading-like` blocks. The spec exercises the `core/table-of-contents` block across editor and frontend contexts, verifying:
- Real-time heading synchronization in the editor
- Nested list structure preservation across publish
- `Include headings down to level` setting enforcement
- Ordered/unordered list style toggling
- Empty-state notices and absent frontend rendering when no headings exist
- Frontend hash-link navigation, scroll-to-section, and pagination (`page=2`) support
Desired behaviors not yet implemented in trunk (e.g., dynamic frontend rendering that reflects post updates outside the editor) are documented via `test.fixme` to prevent regression once implemented.

## Contribution

Opened and merged by @getdave (co-authored with @scruffian) as part of Gutenberg issue #42229. The PR was focused on establishing automated e2e coverage to support the Table of Contents block’s stabilization efforts. Review was minimal, with no alternative approaches or design debates noted in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

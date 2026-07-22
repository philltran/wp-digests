# #78947: Block bindings : add support to list-item

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @saulyz
- **Labels:** `[Type] Enhancement`, `[Feature] Block bindings`
- **Merged:** [`34d09b7`](https://github.com/WordPress/gutenberg/commit/34d09b7423b17c9b3d0234cdf9f5203ea5116913)
- **Discussion:** [#78947](https://github.com/WordPress/gutenberg/pull/78947) · 1 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Block Bindings now supports the `content` attribute of the `core/list-item` block. Previously, list item content could not be dynamically driven by a binding source. This change registers `content` as a supported attribute via the `block_bindings_supported_attributes` filter, allowing developers to bind list item text to custom sources, pattern overrides, or other dynamic data.

## Impact

- **Plugin & theme developers:** Can now use Block Bindings to dynamically populate `<li>` content. No immediate code changes required for existing sites, but developers building custom binding sources or pattern overrides should verify list item bindings.
- **Site owners:** No direct action required. Existing lists render as before.
- **Nested list caveat:** This PR only enables the binding filter. If a bound list item contains a nested `<ul>`/`<ol>` as raw HTML (not inner blocks), the binding will replace the entire content, including the nested list. Preservation of native inner blocks is handled in a follow-up PR (#78991).

## Technical details

The change introduces `lib/compat/wordpress-7.1/block-bindings.php`, which hooks into `block_bindings_supported_attributes` to append `'content'` to the supported attributes array for `core/list-item`. The compat file is loaded via `lib/load.php`. PHPUnit tests in `phpunit/block-bindings-test.php` verify that bound values replace the default content, that raw nested list markup is replaced, and that post-KSES sanitization correctly handles safe HTML (e.g., `<img>`) while stripping unsafe attributes/markup. The build script `tools/build-scripts/dev.mjs` also updates `execSync` and `spawn` calls to use `shell: true` for consistent command execution.

## Contribution

Opened and merged by @saulyz (sauliusv), with co-authorship from @mirka and @cbravobernal. The PR was intentionally split from a larger effort to keep review scope tight: this commit handles the filter enablement and basic test coverage, while nested-list preservation logic was deferred to a separate PR (#78991). The implementation relied on the existing `block_bindings_supported_attributes` filter rather than introducing a new registration API, keeping the change minimal and backward-compatible.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

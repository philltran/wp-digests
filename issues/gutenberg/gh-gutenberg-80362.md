# #80362: Add wp-theme as a style dependency of wp-components

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @mirka
- **Labels:** `[Type] Enhancement`, `No Core Sync Required`
- **Merged:** [`3d514b4`](https://github.com/WordPress/gutenberg/commit/3d514b465a0cc5d780041be55b158173fa3327ee)
- **Discussion:** [#80362](https://github.com/WordPress/gutenberg/pull/80362) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg plugin now registers `wp-theme` as a direct style dependency of `wp-components` and removes it from `wp-base-styles`. This aligns the CSS dependency graph with the JavaScript package declarations, ensuring that `@wordpress/components` styles correctly load the `--wpds-*` design tokens they consume. The change also matches the stylesheet dependency logic being implemented in WordPress Core.

## Impact

- **Plugin & theme developers:** No immediate breaking changes, but the style dependency graph for `wp-components` and `wp-base-styles` has shifted. If you manually dequeue, override, or reorder these handles, verify that `wp-theme` is still loaded where design tokens are required.
- **Gutenberg plugin users:** Design tokens will now load automatically alongside `wp-components` across admin screens that enqueue it, improving consistency with Core's theming rollout.
- **No action required** for standard installations or plugins that rely on the default enqueue behavior.

## Technical details

The diff modifies `lib/client-assets.php` inside the `gutenberg_register_packages_styles()` function. It extracts the `wp-components` registered style object into `$components_style` and appends `wp-theme` to its `$components_style->deps[]` array, alongside the existing `dashicons` dependency. Simultaneously, it removes `wp-theme` from the dependency array passed to `gutenberg_override_style()` for `wp-base-styles` (which loads `admin-schemes.css`). Previously, `wp-base-styles` pulled in `wp-theme` as a convenience, even though it does not consume design tokens. The new graph ensures the `:root` token block loads whenever `wp-components` is enqueued, which now covers virtually all logged-in wp-admin pages due to `wp-commands` depending on `wp-components`.

## Contribution

Opened and merged by @mirka, with co-authors @ciampo, @t-hamano, and @aduth. The PR was discussed in the context of aligning Gutenberg's style graph with upstream Core PR #12560. @t-hamano raised a question about whether custom admin screens would need to explicitly enqueue `wp-theme` for design tokens, and @aduth clarified that while explicit enqueueing is currently required for non-default screens, the dependency chain (`wp-commands` → `wp-components` → `wp-theme`) effectively covers all logged-in admin pages. The approach was accepted without alternative implementations.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

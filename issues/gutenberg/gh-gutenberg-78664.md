# #78664: Theme: Provide design-system token defaults without a runtime `<ThemeProvider>`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Enhancement`, `[Package] Components`, `No Core Sync Required`, `[Package] Theme`
- **Merged:** [`3d1cf3c`](https://github.com/WordPress/gutenberg/commit/3d1cf3c1d62b629ccc0c60349fc83cf69a198cb3)
- **Discussion:** [#78664](https://github.com/WordPress/gutenberg/pull/78664) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg package now ships a prebuilt `design-tokens.css` stylesheet that defines the `:root` design-system token block (`--wpds-*`) on every admin page and inside the editor iframe. This eliminates the previous requirement for a runtime `<ThemeProvider>` to establish default token values, ensuring components always resolve to sensible defaults even when rendered outside a React provider context. The legacy `--wp-components-*` CSS custom properties now chain through `--wpds-*` tokens before falling back to hardcoded values, unifying the fallback resolution path.

## Impact

- **Plugin & theme developers:** No action required. Components consuming `--wp-components-*` or `--wpds-*` will now reliably resolve to design-system defaults without needing a surrounding `<ThemeProvider>`.
- **Build tooling:** If you use `postcss-ds-token-fallbacks` or similar build-time token injection, the fallback chain now explicitly references `--wpds-*` tokens before the hardcoded hex. This is compatible but may slightly alter generated CSS output.
- **No breaking changes.** The `<ThemeProvider>` API remains unchanged; it is still used for runtime theme overrides, but is no longer required for baseline token availability.

## Technical details

- Registers a new PHP style handle `wp-theme` in `lib/client-assets.php`, pointing to `build/styles/theme/design-tokens.css` (and its `.min`/`.rtl` variants).
- Adds `wp-theme` as a dependency to `wp-base-styles` and to `$wp_edit_blocks_dependencies`, guaranteeing the `:root` token block loads before any consuming stylesheets in both the admin and the editor iframe.
- Updates `packages/components/src/utils/colors-values.js` (mirrored in `theme-variables.scss`) to change legacy fallbacks from direct hex values to nested `var()` chains. For example:
  ```css
  /* Before */
  background: var(--wp-components-color-background, #fff);
  
  /* After */
  background: var(--wp-components-color-background, var(--wpds-color-background-surface-neutral-strong, #fff));
  ```
- The diff confirms the author initially considered a `crossesPortalBoundary` prop and per-instance `<style>` emission, but benchmarking revealed negligible real-world gains against added complexity, so those were dropped before merge. The final implementation relies purely on static stylesheet delivery.

## Contribution

The PR evolved through several iterations after its initial submission. An early revision included a per-instance `<style>` optimization and a `crossesPortalBoundary` prop to handle portal overlays, but benchmarking showed unmeasurable performance gains against meaningful added complexity. The author and reviewers converged on a simpler "static for defaults, runtime for diffs" model, delivering the tokens via a prebuilt stylesheet enqueued through PHP. The author noted this was their first contribution to the PHP + stylesheets side of the project, prompting extra review caution on the asset registration logic.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

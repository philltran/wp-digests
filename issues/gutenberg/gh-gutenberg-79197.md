# #79197: Show the admin bar in the Post and Site Editor by default

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Package] Editor`, `[Package] Edit Post`, `[Type] Feature`, `[Package] Interface`, `[Package] Edit Site`
- **Merged:** [`112e09d`](https://github.com/WordPress/gutenberg/commit/112e09d91f3481d18b0e32f8308f67f9d2be7c9b)
- **Discussion:** [#79197](https://github.com/WordPress/gutenberg/pull/79197) · 28 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The Post and Site Editor now render the WordPress admin bar by default, only hiding it when Distraction-free mode is active alongside Fullscreen mode (Post Editor) or just Distraction-free mode (Site Editor). This change graduates the `gutenberg-omnibar` experiment from opt-in gating to permanent default behavior and replaces the previous site-icon navigation hub with an explicit back button in the editor canvas.

## Impact

- **Plugin & theme developers**: No code changes required. The editor UI now consistently shows the admin bar by default, which may affect how custom menus or toolbar overlays are positioned.
- **Developers relying on `window.__experimentalAdminBarInEditor`**: This flag is no longer needed for this behavior; it is now the standard. Relying on it in new code is discouraged.
- **Site owners & editors**: See the admin bar present when editing posts or sites, except when distraction-free/fullscreen modes are both enabled.

## Technical details

The PR removes the experiment gating logic entirely from `lib/experimental/omnibar/load.php`, where `gutenberg_enable_omnibar_experiment()` previously injected `window.__experimentalAdminBarInEditor = true` and applied the `has-admin-bar-in-editor` body class.
- In `lib/experimental/pages/site-editor.php`, a new `gutenberg_site_editor_enable_admin_bar()` callback is registered on `site-editor-v2_init` to explicitly enqueue admin bar assets and render it above the editor canvas.
- The frontend `packages/boot/src/components/canvas/back-button.tsx` was simplified: conditional rendering based on `__experimentalAdminBarInEditor` was removed, along with `motion`-based animations and hover variants, leaving a static chevron back button.
- Layout positioning in `packages/boot/src/components/root/style.scss` now uses the CSS variable `var(--wp-admin--admin-bar-height, 0)` for top offsets and viewport height calculations instead of hardcoded `top: 0`.
- The experiment metadata in `lib/experimental/experiments/load.php` was updated from 'Omnipresent Toolbar' to 'Toolbar UI refresh'.

## Contribution

Opened by @fushar and merged following review. Co-authored by @scruffian, @youknowriad, @mtias, @jasmussen, and others. Discussion noted that the legacy `gutenberg-omnibar` toggle should be removed from Core's experiments page in a follow-up, and clarified that a related Site Editor V2 site logo sizing bug was being addressed separately in Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

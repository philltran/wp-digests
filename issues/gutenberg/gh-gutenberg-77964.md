# #77964: Add experiment to show admin bar in Post and Site Editor

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Package] Edit Post`, `[Type] Experimental`, `[Package] Edit Site`
- **Merged:** [`8f9c4b3`](https://github.com/WordPress/gutenberg/commit/8f9c4b3966068a78c37b01904e84c7c31d691e73)
- **Discussion:** [#77964](https://github.com/WordPress/gutenberg/pull/77964) · 18 comments · 0 reactions

## Summary

A new Gutenberg experiment called `gutenberg-admin-bar-in-editor` makes the WordPress Admin Bar visible inside the fullscreen Post and Site Editors. The admin bar has always been rendered in the page HTML; the fullscreen editor chrome simply covered it. Enabling the experiment adds the `.is-admin-bar-in-editor-enabled` class to `<body>`, triggering CSS that repositions the editor to reveal the admin bar. The top-left site icon is simultaneously replaced with an explicit back button, giving editors a clearer exit path. Distraction-free mode suppresses the bar as expected. This is tracked in Trac #65091.

## Impact

**Plugin & theme developers**
- No action required. The change is entirely opt-in and gated behind Gutenberg > Experiments > **Toolbar in editor**.
- No new PHP hooks, JS filters, or public APIs are introduced.
- CSS that styles the admin bar area in the editor (e.g., custom `#wpadminbar` rules) may interact with the new layout adjustments when the experiment is active; test under the `.is-admin-bar-in-editor-enabled` body class if your plugin/theme targets that element.

**Site owners / content editors**
- Only relevant when the Gutenberg plugin is installed and the experiment is manually enabled.

**Hosting & platform**
- No server-side changes.

**Headless & REST consumers**
- Not affected.

**Note:** Admin color scheme support was drafted but reverted from this PR; it will land in a separate pull request (#78331).

## Technical details

**Experiment registration**
- A new experiment with the key `gutenberg-admin-bar-in-editor` is registered. Boot-time JS (`build/modules/boot/index.min.js`, +309 B) handles activation.
- When the experiment is active the editor adds `.is-admin-bar-in-editor-enabled` to `<body>`.

**New stylesheet assets (four files, all new)**
```
build/styles/edit-post/experimental-admin-bar-in-editor.css        (~551 B)
build/styles/edit-post/experimental-admin-bar-in-editor-rtl.css    (~550 B)
build/styles/edit-site/experimental-admin-bar-in-editor.css        (~502 B)
build/styles/edit-site/experimental-admin-bar-in-editor-rtl.css    (~502 B)
```
These stylesheets scope all rules under `.is-admin-bar-in-editor-enabled` and adjust the editor container's top offset to account for the `32px` admin bar height.

**Existing stylesheet modifications**
- `build/styles/edit-post/style.css` +290 B (+8.25%) — supporting layout rules added unconditionally for the Post Editor.
- `build/styles/edit-site/style.css` +215 B (+1.02%) — same for the Site Editor.

**JS packages changed**
- `build/scripts/edit-post/index.min.js` (+213 B) — replaces the top-left site icon with an explicit back button when the experiment is enabled.
- `build/scripts/edit-site/index.min.js` (+142 B) — same replacement in the Site Editor.

**Distraction-free interaction**
When the user toggles Distraction-free mode, the admin bar is hidden again, consistent with the existing distraction-free behavior.

**RTL support** is included via the separate `-rtl` CSS variants.

## Contribution

@fushar opened and authored the PR, noting use of Codex for implementation assistance. The PR was merged at commit `8f9c4b3`. One notable scope decision during review: admin color scheme support (ensuring the admin bar's color tokens matched each WP admin color scheme) was implemented but then reverted from this PR to keep the changeset focused; it is tracked separately in PR #78331. The underlying UX rationale — reducing user disorientation in the fullscreen editor — is documented in Trac ticket #65091.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

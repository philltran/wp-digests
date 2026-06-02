# #78493: wp-build: Fix black flash on wp-admin pages before hydration

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @itzmekhokan
- **Labels:** `[Type] Bug`, `Backport to WP Minor Release`
- **Merged:** [`6343b59`](https://github.com/WordPress/gutenberg/commit/6343b59e9e95c6dad06b9242a8e597c6399dd15b)
- **Discussion:** [#78493](https://github.com/WordPress/gutenberg/pull/78493) · 5 comments · 1 reactions

## Summary

A misapplied design token in the `@wordpress/wp-build` critical CSS caused a brief black flash on wp-admin integrated boot pages (such as the Connectors screen) before `@wordpress/boot` hydrated the React app. The `#wpwrap` element had its `background` set to `var(--wpds-color-fg-content-neutral, #1e1e1e)` — a **foreground** WPDS token that resolves to near-black — rather than a background token. The fix removes the `background` declaration from `#wpwrap` entirely, leaving `overflow-y: auto` in place and relying on the existing `body { background: #fff; }` for the pre-hydration paint. Generated `page-wp-admin.php` files in WordPress Core are produced from this template, so the fix must land in `wp-build` rather than in `wordpress-develop` directly.

## Impact

- **Site owners / admins:** The black flash before the Connectors screen (and any other wp-admin integrated boot page) renders is eliminated. No configuration changes needed.
- **Plugin & theme developers:** No action required. If you consume generated `page-wp-admin.php` output from `@wordpress/wp-build`, rebuild from the updated template. Custom pages that inline their own critical CSS with `#wpwrap { background: ... }` should audit whether they are also using a foreground token as a background.
- **Hosting & platform:** Tagged `Backport to WP Minor Release` (tracking Core Trac [#65247](https://core.trac.wordpress.org/ticket/65247)); the fix will ship in an upcoming WordPress minor. No server-side changes required.
- **Headless / REST consumers:** Not affected.

## Technical details

**File changed:** `packages/wp-build/templates/page-wp-admin.php.template`

The inlined critical `<style>` block previously set a `background` on `#wpwrap` to a foreground design token. Because this CSS is rendered before `@wordpress/boot` attaches and `.boot-layout` styles are applied, the element briefly painted near-black.

**Before:**
```css
#wpwrap {
    background: var(--wpds-color-fg-content-neutral, #1e1e1e);
    overflow-y: auto;
}
```

**After:**
```css
#wpwrap {
    overflow-y: auto;
}
```

The `overflow-y: auto` is retained — it was added in PR #73977 to fix admin-menu scrolling at high zoom levels and is unrelated to the color bug.

Pre-hydration background now inherits from the `body` rule already present lower in the same template:
```css
body {
    background: #fff;
}
```

After hydration, `.boot-layout__canvas` / `.boot-layout__stage` apply their own surface tokens (`--wpds-color-bg-surface-neutral`), so the removal has no visible effect on the fully-rendered state.

A `CHANGELOG.md` entry was added to `packages/wp-build` in a follow-up PR (#78807).

## Contribution

Opened and authored by @itzmekhokan. The initial implementation replaced the foreground token with `var(--wpds-color-bg-surface-neutral-weak, #f0f0f0)`, but @t-hamano noted during review that the actual rendered content area uses `--wpds-color-bg-surface-neutral` and suggested the cleaner fix was to remove the background declaration entirely. @itzmekhokan updated the PR accordingly. @westonruter reviewed the approach in the context of the now-closed `wordpress-develop` PR #11892, confirming the fix must originate in the `wp-build` template rather than in generated Core files. @simison filed the missing changelog entry as a separate PR (#78807) after merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

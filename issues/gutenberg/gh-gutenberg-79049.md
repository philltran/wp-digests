# #79049: Omnipresent toolbar: show site icon instead of dashicon if set

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @fushar
- **Labels:** `[Type] Enhancement`
- **Merged:** [`8eb6461`](https://github.com/WordPress/gutenberg/commit/8eb646185862f70192c8e1ceea1c3dbb08514bb2)
- **Discussion:** [#79049](https://github.com/WordPress/gutenberg/pull/79049) · 2 comments · 0 reactions

## Summary

The Gutenberg plugin’s omnipresent toolbar experiment now replaces the default admin bar site menu icon with the actual site icon when one is configured. This standardizes visual identity across frontend and editor views, resolving confusion from previously switching between generic dashicons based on context. It improves mobile viewport identification without altering the node label or core admin bar structure.

## Impact

- **Gutenberg users & editors**: Affects only the “Omnipresent Toolbar” (`gutenberg-omnibar`) experiment. No action required; site icons now appear automatically when set via Settings > General.
- **Theme/Plugin developers**: No breaking changes or API deprecations. The `site-name` admin bar node is re-rendered with a `.has-site-icon` modifier class and an explicit `<img>` element instead of a CSS pseudo-element. Custom themes targeting `#wp-admin-bar-site-name .ab-item:before` for styling will need to account for the new inline image or the `content: none` rule.
- **Hosting/Platform**: No action required. Behavior is gated behind the Gutenberg experiment flag and requires no server configuration.

## Technical details

- **File**: `lib/experimental/omnibar/load.php` (Gutenberg repository)
- **Behavior change**: The diff introduces `gutenberg_omnibar_site_icon()` hooked to `admin_bar_menu` at priority 31. It gates execution on `is_admin_bar_showing()` and `gutenberg_is_experiment_enabled( 'gutenberg-omnibar' )`, retrieves the `'site-name'` node, and verifies a URL exists via `get_site_icon_url( 64 )`. If present, it re-adds the node with an inline `<img class="site-icon">` prefixed to the title.
- **Styling**: `gutenberg_omnibar_site_icon_styles()` hooks into `wp_enqueue_scripts` and `admin_enqueue_scripts`, using `wp_add_inline_style( 'admin-bar', $css )` to suppress the default dashicon (`content: none`) on `.has-site-icon > .ab-item:before`. The icon is styled at 20×20px with a `#f0f0f1` background and `border-radius: 2px`. A media query for `max-width: 782px` switches to absolute positioning and scales it to 28×28px.
- **Pattern shift**:
  ```diff
  // Before (implicit dashicon rendered by core admin-bar.css)
  $node->title = 'Site Name';

  // After (explicit inline image, gated by experiment)
  $meta['class'] = isset( $meta['class'] ) ? trim( $meta['class'] . ' has-site-icon' ) : 'has-site-icon';
  $wp_admin_bar->add_node(
    array(
      'id'   => 'site-name',
      'title'=> '<img class="site-icon" src="' . esc_url( $site_icon_url ) . '" alt="" />' . $node->title,
      'meta' => $meta,
    )
  );
  ```

## Contribution

Merged by @fushar with co-authored commits from @StevenDufresne and @arthur791004 (commit `8eb6461`). The PR addresses visual inconsistency in the omnipresent toolbar experiment (linked to issue #79036). Review focused on UI consistency across editor themes and mobile viewports; flaky e2e test reports were noted but did not impact the merge decision. No alternative approaches or rejected implementations were documented in the discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

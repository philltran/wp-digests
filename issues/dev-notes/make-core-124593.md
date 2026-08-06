# Registering and rendering SVG icons in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aki Hamano
- **Published:** 2026-07-24
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/24/registering-and-rendering-svg-icons-in-wordpress-7-1/](https://make.wordpress.org/core/2026/07/24/registering-and-rendering-svg-icons-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces a formal public API for managing and rendering SVG icons, moving beyond the 7.0 built-in set. Developers can now register custom icon collections, add individual icons via inline markup or file paths, render them server-side, and expose them to the editor and REST API. This standardizes icon handling across plugins, themes, and core blocks.

## Impact

- **Plugin & theme developers:** Can register custom icons using `wp_register_icon_collection()` and `wp_register_icon()`. Must ensure SVGs only contain `<svg>`, `<path>`, and `<polygon>` elements with allowed attributes, as `wp_kses` strips everything else.
- **Block developers:** The core Icon block now groups icons by collection in its picker, adds flip/rotate toolbar controls, defaults to `core/info`, and delegates server rendering to `wp_get_icon()`.
- **REST consumers:** New read-only endpoints (`/wp/v2/icon-collections`, `/wp/v2/icons`) expose icon metadata and markup. Access requires `edit_posts` capability or equivalent.
- **Styling note:** `wp_get_icon()` output lacks `fill="currentColor"` on the root `<svg>` by default, so standalone icons render in black unless styled via CSS or registered with `fill="currentColor"` on inner shapes.

## Technical details

Registration runs on the `init` hook. `wp_register_icon_collection( $name, $args )` accepts a slug and `label`/`description`. `wp_register_icon( $name, $args )` accepts `collection/icon-name` formatted slugs and either `content` (inline SVG string) or `file_path` (lazily loaded). Both functions return boolean success and trigger `_doing_it_wrong()` on validation failure.
Rendering uses `wp_get_icon( $name, $args )`, which accepts `size`, `class`, and `label` parameters. The function returns sanitized SVG markup.
REST endpoints are read-only and require authentication with `edit_posts` capability:
- `GET /wp/v2/icon-collections` and `GET /wp/v2/icon-collections/<collection>`
- `GET /wp/v2/icons`, `GET /wp/v2/icons/<collection>`, and `GET /wp/v2/icons/<collection>/<name>`
Query parameters `search` and `collection` filter the icon list.

```php
// Registration
wp_register_icon_collection( 'my-plugin', [ 'label' => 'My Icons' ] );
wp_register_icon( 'my-plugin/star', [ 'label' => 'Star', 'content' => '<svg>...</svg>' ] );

// Rendering
echo wp_get_icon( 'my-plugin/star', [ 'size' => 32, 'class' => 'my-icon' ] );
```

## Contribution

The feature evolved from the 7.0 built-in icon set into a formalized API, with review by @tyxla. The implementation deliberately kept the `wp_kses` allowlist conservative to prevent XSS, deferring broader attribute support to a separate Gutenberg PR. The lazy-loading behavior for `file_path` was chosen to avoid filesystem I/O during registration, with errors surfacing only when content is first requested.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

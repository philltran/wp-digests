# #77260: Icons: Add APIs for collection and icon registration

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `Needs Dev Note`, `[Block] Icon`, `[Feature] Icons`
- **Merged:** [`a839793`](https://github.com/WordPress/gutenberg/commit/a83979338de7dfb218ab1ac36750f7b631e3a1d8)
- **Discussion:** [#77260](https://github.com/WordPress/gutenberg/pull/77260) · 15 comments · 2 reactions
- **Usefulness:** 4/5

## Summary

This change introduces a formalized PHP API for registering and managing SVG icons within collections. It shifts icon registration from internal class constructors to public wrapper functions, enabling plugin and theme developers to expose their own icon sets through a unified, namespaced system. Icons are now uniquely identified by `{collection-slug}/{icon-slug}` and can be queried via new REST endpoints, establishing the backend foundation for upcoming block editor icon picker features.

## Impact

- **Plugin & Theme Developers:** Can now officially register custom SVG collections and icons using `wp_register_icon_collection()` and `wp_register_icon()`. While there are no immediate breaking changes, developers should adopt this pattern to ensure namespace isolation (e.g., avoiding conflicts between a theme's `star` and core's `star`).
- **Block Developers:** Gain access to predictable namespaces for querying registered SVGs via REST. This prepares the ecosystem for filtered icon pickers and future `theme.json` extensions that reference specific collections.
- **Core/Platform Teams:** The singleton `WP_Icon_Collections_Registry` now handles lifecycle management, replacing hardcoded constructor registrations with a centralized `gutenberg_register_default_icons()` function.

## Technical details

The PR introduces `WP_Icon_Collections_Registry` as a singleton to manage collection metadata and extends `WP_Icons_Registry_Gutenberg` to accept a collection slug parameter during registration. The diff moves core icon definitions out of the constructor into a dedicated `gutenberg_register_default_icons()` callback, establishing a clear initialization pattern.

New public wrapper functions handle lifecycle management:
```php
add_action( 'init', function () {
    wp_register_icon_collection( 'my-icons', array(
        'label' => 'My Icons',
        'description' => 'Custom brand icons.',
    ) );
    wp_register_icon( 'my-icons/star', array(
        'label' => 'Star',
        'content' => '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="..."/></svg>',
    ) );
} );
```

The `WP_REST_Icons_Controller_Gutenberg` now routes `/wp/v2/icons/<namespace>` to filter results by collection slug, while `/wp/v2/icons/<namespace>/<name>` retains single-item lookup behavior. Deleting a collection implicitly purges all belonging icons from the registry.

## Contribution

Merged on commit `a839793` by @t-hamano, with co-authors @mcsf and @mamaduka. Part of a broader epic (#75715) to overhaul the icon system, this PR established the PHP backbone before frontend filtering UI work was implemented. Review focused on balancing explicit collection registration against convenience aliases; the final diff committed to strict `{namespace}/{slug}` identifiers to prevent global namespace collisions. The author noted AI assistance was used for refactoring boilerplate and commit messages, with manual review ensuring architectural accuracy.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

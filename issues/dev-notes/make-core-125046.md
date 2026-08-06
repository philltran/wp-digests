# Filtering Site Editor Screens in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** André Maneiro
- **Published:** 2026-07-31
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/31/filtering-site-editor-screens-in-wordpress-7-1/](https://make.wordpress.org/core/2026/07/31/filtering-site-editor-screens-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 introduces four new filters that allow developers to programmatically configure the DataViews and DataForm components powering Site Editor screens for pages, templates, template parts, and patterns. These hooks expose configuration for default view layouts, sidebar filters, field visibility, sort orders, and Quick Edit forms, replacing the need to modify core UI code or rely on undocumented internals.

## Impact

- **Theme & plugin developers:** Can now customize Site Editor listing screens and Quick Edit forms via `add_filter()` without overriding templates or hacking JavaScript.
- **Agency & platform teams:** Will benefit from standardized, filter-driven configuration that persists across updates, though no immediate action is required unless custom UI tweaks are in use.
- **No breaking changes or deprecations** are introduced; existing Site Editor screens continue to function as before.

## Technical details

The change adds four filters named `get_entity_view_config_posttype_page`, `get_entity_view_config_posttype_wp_template`, `get_entity_view_config_posttype_wp_template_part`, and `get_entity_view_config_posttype_wp_block`. Each receives a configuration object with a `merge()` method and exposes four configurable keys: `default_view` (layout, sort, fields), `default_layouts`, `view_list` (sidebar filters like “All”, “Published”), and `form` (Quick Edit field order/visibility). Example usage:

```php
add_filter( 'get_entity_view_config_posttype_page', function( $data ) {
    $patch = array(
        'default_view' => array(
            'type'   => 'grid',
            'sort'   => array( 'field' => 'title', 'direction' => 'asc' ),
            'fields' => array( 'date' ),
        ),
    );
    $data->merge( $patch, 1 );
    return $data;
} );
```

These filters feed directly into the `DataViews` and `DataForm` React components used in the Site Editor, centralizing configuration that previously lived in disparate JavaScript modules.

## Contribution

Tracked under ticket #76544 as part of the broader DataViews and DataForm iteration for 7.1, the feature was developed to consolidate Site Editor configuration into a single, filter-driven API. Review was provided by @ntsekouras and @priethor, with the implementation focusing on exposing the existing React component configs rather than rebuilding the UI layer. Future iterations plan to extend this pattern to other entities and unify the editor inspector with these same filters.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# HTML API: Respect namespace in open element lookup.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-06-29
- **Commit:** [`f149add5b5`](https://github.com/WordPress/wordpress-develop/commit/f149add5b5196da9c23c93bb9fdea6c787b55c21)
- **Usefulness:** 3/5

## Summary

The HTML API's open element stack lookup now explicitly validates the XML namespace before matching a tag name, preventing foreign elements (such as MathML or SVG) from incorrectly satisfying checks for native HTML elements. This fixes a parsing state machine bug where a `<template>` tag inside a `<math>` block could prematurely close the main document's template context. The change ensures that DOM tree structure and breadcrumb tracking remain accurate when processing mixed-content fragments containing namespace-scoped markup.

## Impact

- **Plugin & Theme Developers / Block Authors**: No immediate action required. The fix addresses internal parser state within the core HTML API rather than modifying public-facing hooks or REST endpoints.
- **Custom DOM Processors / Headless Consumers**: Affected if you are building on top of `WP_HTML_Processor` or manually manipulating open element stacks around mixed namespaces. The change ensures `contains()` lookups correctly distinguish between HTML and foreign elements, preventing unintended breadcrumb shifts.
- **Hosting & Platform Teams**: No migration or configuration changes needed; this is a targeted bug fix to the core tokenizer/state machine.

## Technical details

The unified diff modifies `WP_HTML_Open_Elements::contains()` in `src/wp-includes/html-api/class-wp-html-open-elements.php`. Previously, the method returned `true` if `$node_name === $item->node_name`, ignoring the element's namespace. It now requires `'html' === $item->namespace && $node_name === $item->node_name`, ensuring only native HTML elements trigger a match in the open stack.

```php
// Before
if ( $node_name === $item->node_name ) {
    return true;
}

// After
if ( 'html' === $item->namespace && $node_name === $item->node_name ) {
    return true;
}
```
A corresponding test in `tests/phpunit/tests/html-api/wpHtmlProcessor.php` (`test_unmatched_template_closer_after_mathml_template_is_ignored()`) verifies that a `<template>` tag nested inside `<math>` does not satisfy HTML template handling, preserving the expected breadcrumb array: `['HTML', 'BODY', 'MATH', 'TEMPLATE', 'MI', 'C', '#text']`.

## Contribution

Authored by Jon Surrell (PR #12353, ticket #65372) and merged on 2026-06-29. The implementation focused on tightening the `contains()` method in the HTML API's open element tracker to explicitly validate the `'html'` namespace before returning a match. Review credits were extended to dmsnell for validating the edge case with foreign namespace templates.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# HTML API: Update SELECT element parsing according to specification.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-07-10
- **Commit:** [`234ae9bd59`](https://github.com/WordPress/wordpress-develop/commit/234ae9bd59e7c7a101725add25e955a354ae6017)
- **Usefulness:** 3/5

## Summary

WordPress's HTML API parser has been updated to align with recent HTML Standard changes regarding customizable `<select>` elements. The legacy "select scope" concept and dedicated "in select" insertion modes have been removed, replaced with standard stack-based scope checks. This adjustment ensures tokenization and insertion logic for `<select>`, `<option>`, and `<optgroup>` tags match modern browser behavior, particularly in nested or fragment parsing scenarios.

## Impact

- **Plugin & theme developers:** No immediate code changes required. The `WP_HTML_Tag_Processor` and `WP_HTML_Processor` public interfaces remain stable.
- **Advanced HTML API consumers:** `WP_HTML_Open_Elements::has_element_in_select_scope()` is deprecated in WordPress 7.1.0 and now always returns `false`. Callers should remove or refactor any usage.
- **Parser behavior:** Nested `<select>` and `<input>` tokens are now handled via `has_element_in_scope( 'SELECT' )` checks rather than dedicated insertion mode switches, which may subtly alter parsing order for complex or malformed HTML fragments.
- No configuration, migration, or theme/plugin updates are required to adopt this change.

## Technical details

The diff updates three core HTML API files to strip out legacy `<select>` parsing modes and standardize scope resolution:
- **`src/wp-includes/html-api/class-wp-html-open-elements.php`**: Adds `SELECT` to the `specific_scope`, `scope`, `list_item_scope`, and `button_scope` termination groups. The `has_element_in_select_scope()` method is deprecated and hardcoded to return `false`. `SELECT` is also registered in `after_element_push()` and `after_element_pop()`.
- **`src/wp-includes/html-api/class-wp-html-processor.php`**: Deletes the `step_in_select()` and `step_in_select_in_table()` private methods. The main `step()` switch no longer routes to these modes. Instead, `step_in_body()` now handles `+SELECT`, `+OPTION`, `+OPTGROUP`, and `+INPUT` tokens by invoking `$this->state->stack_of_open_elements->has_element_in_scope( 'SELECT' )`, applying `generate_implied_end_tags()`, or popping the stack as dictated by the updated spec. A docblock note clarifies that the parser does not yet implement the "maybe clone an option into selectedcontent" algorithm.
- **`src/wp-includes/html-api/class-wp-html-processor-state.php`**: Marks `INSERTION_MODE_IN_SELECT` and `INSERTION_MODE_IN_SELECT_IN_TABLE` constants as deprecated in docblocks.

## Contribution

Opened as PR #9298 and merged on 2026-07-10 as commit `234ae9bd59`, this change was authored by Jon Surrell with review contributions from Douglas Snell. It addresses trac ticket #63736 by directly implementing the HTML Standard's removal of "select scope" and dedicated `<select>` insertion modes, consolidating the processor's token routing into unified stack scope checks.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

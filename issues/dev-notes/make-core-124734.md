# Dev Chat summary: July 21, 2026

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Jb Audras
- **Published:** 2026-07-21
- **Tags:** `Devchat`, `General`, `Summaries`, `7.0.2`, `7.1`, `core`, `dev chat`
- **Link:** [https://make.wordpress.org/core/2026/07/21/summary-dev-chat-july-21-2026/](https://make.wordpress.org/core/2026/07/21/summary-dev-chat-july-21-2026/)
- **Usefulness:** 4/5

## Summary

WordPress Core is introducing opt-in interactive, runnable PHP code examples in the developer documentation for version 7.1. Instead of static indented PHPDoc snippets, documentation authors can now use ` ```php interactive ` code fences to embed live Playground-powered examples that execute directly in the browser. This change modernizes how Core APIs are demonstrated and removes the friction of manually testing code examples.

## Impact

- **Documentation authors & contributors:** Must adopt the new ` ```php interactive ` syntax for examples intended to be runnable; existing static snippets remain unchanged and require no migration.
- **Plugin & theme developers:** No immediate code changes or configuration updates required. The enhancement improves API discoverability and testing workflows but does not alter runtime behavior.
- **Platform & hosting teams:** No action required; execution is entirely client-side via WordPress Playground.
- **Note:** This is a documentation-site enhancement, not a core PHP/JS runtime change.

## Technical details

The feature spans two coordinated repositories: `phpdoc-parser` (PR #258) and `wporg-developer` (PR #567). The parser now recognizes the `interactive` modifier in PHPDoc code fences, extracting the snippet for execution. The documentation site renders these as Playground-powered widgets. A standard static example:

```php
$p = WP_HTML_Processor::create_fragment( "<div class='free &lt;egg&lt;\tlang-en'>" );
$p->next_tag();
foreach ( $p->class_list() as $class_name ) {
    echo "{$class_name} ";
}
```

becomes interactive when wrapped in the new fence:

```php interactive
$p = WP_HTML_Processor::create_fragment( "<div class='free &lt;egg&lt;\tlang-en'>" );
$p->next_tag();
foreach ( $p->class_list() as $class_name ) {
    echo "{$class_name} ";
}
```

Execution is entirely delegated to the WordPress Playground environment; no server-side changes or new PHP functions are introduced.

## Contribution

Jon Surrell and Adam Zielinski are leading the initiative, building on earlier documentation parsing improvements by Jon Surrell and Daniel Snell during the 7.0 cycle. The team deliberately scoped the feature to an opt-in syntax rather than a blanket migration of all existing examples, allowing gradual adoption ahead of the 7.1 release. The parser and documentation rendering changes are tracked in separate PRs to allow independent review and iteration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# HTML API: Discourage public use of the `step()` method.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-06-30
- **Commit:** [`aa4e8ec2fe`](https://github.com/WordPress/wordpress-develop/commit/aa4e8ec2fe1dd7c95e376cc87ba60e856eb2b81a)
- **Usefulness:** 3/5

## Summary

The `WP_HTML_Processor::step()` method has been formally marked as internal and discouraged for public use via a docblock update. This clarifies that `next_tag()` and `next_token()` are the intended iteration methods, steering developers away from lower-level stepping mechanisms in the HTML API without altering runtime behavior.

## Impact

- **Plugin & theme developers / core contributors**: No code changes required, but direct invocation of `step()` is now officially discouraged. Builders using the HTML API for parsing or block rendering should migrate to `next_tag()` or `next_token()`.
- **Hosting & platform teams**: No action required.
- **Breaking changes**: None. The method remains callable, but its `@access private` marking signals potential future refactoring or removal without prior deprecation notice.

## Technical details

The unified diff modifies the PHPDoc for `WP_HTML_Processor::step()` in `src/wp-includes/html-api/class-wp-html-processor.php`. It injects a clarifying comment stating the method is internal and explicitly references `::next_tag()` and `::next_token()` as the public iteration paths. The docblock also receives an `@access private` attribute, formally demoting its API visibility from a developer-facing standpoint. No signature, logic, or behavioral changes were introduced in this commit.

## Contribution

Authored by Jon Surrell (`jonsurrell`) and merged on 2026-06-30 as part of the HTML API boundary-tightening effort (Props: `dmsnell`). The change originated from PR #12269 and follow-up ticket #64896, focusing on stabilizing public iteration patterns before broader ecosystem adoption.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

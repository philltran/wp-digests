# #79972: Media: Fix fatal error from narrowed create_item_from_url() visibility

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `No Core Sync Required`
- **Merged:** [`15ba6e2`](https://github.com/WordPress/gutenberg/commit/15ba6e2ac525aa0f330d64b7ba8df666903b7b79)
- **Discussion:** [#79972](https://github.com/WordPress/gutenberg/pull/79972) · 6 comments · 2 reactions
- **Usefulness:** 3/5

## Summary

Resolves a fatal error in the standalone Gutenberg plugin caused by an incorrectly scoped REST controller method. The `create_item_from_url()` override in `Gutenberg_REST_Attachments_Controller` was declared `private`, violating PHP's rule that child classes cannot narrow parent method visibility. Matching the parent `protected` scope restores class loading and prevents cascading test suite failures and potential REST 500 errors.

## Impact

- **Plugin & theme developers**: No direct code changes required; the fix is internal to a controller override.
- **Hosting & platform teams / CI maintainers**: Eliminates a fatal error that breaks PHP unit tests and causes 500 errors in e2e and performance suites when running trunk or PR branches.
- **WordPress Core sync**: Marked `No Core Sync Required`; this addresses the standalone package only and does not require a core merge.

## Technical details

The diff modifies `lib/media/class-gutenberg-rest-attachments-controller.php` by changing the visibility of `Gutenberg_REST_Attachments_Controller::create_item_from_url()` from `private` to `protected`. It also removes the explicit `WP_REST_Request $request` type hint to align with the parent signature. Prior to this change, PHP 8's strict visibility rules triggered a fatal error during class autoload, which halted the test suite and disrupted REST attachment endpoints.

```
// Before
private function create_item_from_url( WP_REST_Request $request ) {

// After
protected function create_item_from_url( $request ) {
```

## Contribution

Opened by @adamsilverstein to resolve a regression introduced in #79409. After brief review from @andrewserong and @ramonjd regarding test behavior, the PR was auto-merged to unblock CI on trunk. The commit includes co-authors for the Gutenberg team's contribution process, with the code change solely authored by Silverstein.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

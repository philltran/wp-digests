# #80769: RTC: Disable custom autosave controller when RTC is disabled

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ingeniumed
- **Labels:** `[Type] Bug`, `[Feature] Real-time Collaboration`, `No Core Sync Required`
- **Merged:** [`b912823`](https://github.com/WordPress/gutenberg/commit/b912823ddad2bd131c8cbf4a250ff71b4da9e710)
- **Discussion:** [#80769](https://github.com/WordPress/gutenberg/pull/80769) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug where Gutenberg’s custom autosave REST controller executed even when real-time collaboration (RTC) was disabled, causing stale preview links when reverting post titles. The change adds an early return to `Gutenberg_REST_Autosaves_Controller::create_item()` that delegates to Core’s `WP_REST_Autosaves_Controller::create_item()` when `wp_is_collaboration_enabled()` returns false, and updates `gutenberg_override_autosaves_rest_controller()` to only register the custom controller when RTC is active.

## Impact

- **Plugin & theme developers**: No direct API changes. Autosave behavior for post types with RTC disabled now strictly follows Core’s default logic, eliminating unexpected revision creation or preview link caching.
- **Site owners & editors**: Resolves stale preview links when editing titles and reverting them while RTC is turned off in Settings → Writing.
- **No action required**: The fix is transparent and aligns with expected Core behavior. Existing codebases do not need migration or configuration changes.

## Technical details

- In `lib/experimental/collaboration/class-gutenberg-rest-autosaves-controller.php`, `create_item()` now checks `wp_is_collaboration_enabled()` and `wp_is_post_type_collaboration_disabled()` before executing RTC-specific logic. When either condition is false, it immediately returns `parent::create_item( $request )`.
- In `lib/experimental/collaboration/rest-api.php`, `gutenberg_override_autosaves_rest_controller()` now conditionally sets `autosave_rest_controller_class` only when `wp_is_collaboration_enabled()` is true, preventing the custom class from being registered unnecessarily.
- The diff removes redundant conditional branches inside `create_item()` that previously attempted to gate RTC behavior, consolidating control to the registration layer and a single early-return guard.
- PHPUnit tests in `phpunit/tests/collaboration/restAutosavesController.php` verify controller selection and delegation. E2E tests in `test/e2e/specs/editor/collaboration/collaboration-disabled-post-type-autosave.spec.ts` validate direct draft updates and revision creation for collaboration-disabled post types.

## Contribution

Opened by @ingeniumed to address a regression in autosave preview behavior when RTC is disabled. During review, @Mamaduka and @chriszarate suggested avoiding the class override entirely when RTC is off. @ingeniumed revised the registration logic in `gutenberg_override_autosaves_rest_controller()` to conditionally apply the custom controller, while retaining the early-return guard in `create_item()` as a defensive fallback. The PR merged with updated tests covering both global and per-post-type RTC toggles.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

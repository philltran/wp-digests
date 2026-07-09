# #79591: RTC: Fix autosave update with no content

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Bug`, `[Feature] Real-time Collaboration`, `No Core Sync Required`
- **Merged:** [`f93481c`](https://github.com/WordPress/gutenberg/commit/f93481cbbfa0c3c14f6026f3249d668910dec639)
- **Discussion:** [#79591](https://github.com/WordPress/gutenberg/pull/79591) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Real-Time Collaboration (RTC) previously triggered a false-positive “There is an autosave of this post that is more recent than the version below” notice when an autosave fired without changing any post content or revisioned meta. This fix prevents the REST autosaves controller from persisting redundant revisions for no-op payloads, eliminating spurious warnings caused purely by timestamp increments.

## Impact

['- **Multi-user editors & agency platform teams**: Will no longer see disruptive stale-autosave notices after routine autosaves or non-content changes (e.g., taxonomy updates) that do not alter the post body.', '- **Plugin & theme developers**: No breaking changes. The REST endpoint behavior remains backwards-compatible; the modification only gates revision creation server-side during RTC sessions and does not affect third-party autosave consumers.', '- **Configuration/Migration**: No action required. Existing editor workflows, saved drafts, and revisions remain unaffected.']

## Technical details

The unified diff modifies `Gutenberg_REST_Autosaves_Controller::create_item()` in `lib/compat/wordpress-7.0/class-gutenberg-rest-autosaves-controller.php`. A new private method, `is_redundant_autosave( $post, $post_data, $meta, $user_id )`, is introduced to evaluate whether an incoming autosave payload matches the parent draft. The method explicitly compares revisioned post fields (`title`, `content`, `excerpt`) and revisioned meta keys (using `get_metadata_raw()` alongside `_wp_post_revision_fields()`), while deliberately ignoring non-revisioned meta like `_crdt_document`. When a no-op is detected, the controller skips creating a new revision and returns the parent post’s ID as `$autosave_id`, preventing core’s timestamp-based stale detection in `wp-admin/edit-form-blocks.php` from triggering. Unit tests were added to `phpunit/tests/collaboration/restAutosavesController.php` and an e2e spec validates that the spurious notice is removed during headless editing sessions.

## Contribution

Opened by @alecgeatches and reviewed/merged by @maxschmeling (commit `f93481c`). The PR discussion centered on aligning the autosave controller with WordPress’s existing revision diff logic, ensuring that changes to revision-enabled meta (like footnotes) still trigger proper revisions while stripping timestamp-only drifts. Mid-review testing expanded to cover both no-op skipping and legitimate meta-driven revision creation before merge.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

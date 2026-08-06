# #79606: Notes: Email users mentioned in a note

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Feature`, `Backported to WP Core`, `[Feature] Notes`
- **Merged:** [`6dc32fc`](https://github.com/WordPress/gutenberg/commit/6dc32fcd7e4258df7742e0ac661ed6adaaed569c)
- **Discussion:** [#79606](https://github.com/WordPress/gutenberg/pull/79606) · 18 comments · 2 reactions
- **Usefulness:** 4/5

## Summary

This PR adds email notifications for users `@`mentioned in a block comment (note). When a note is created via the REST API, the system parses mention chips from the saved content and sends a localized email to each mentioned user, linking directly to the post editor. This completes the mention feature introduced in #79604 by adding the delivery half, ensuring collaborators are alerted to relevant internal feedback without requiring manual outreach.

## Impact

- **Plugin & theme developers:** No code changes required. The feature is opt-in via the existing `wp_notes_notify` option and respects standard capability checks.
- **Site owners & editors:** Mentioned users will now receive email notifications when added to a note, streamlining internal review workflows.
- **Hosting & platform teams:** No configuration changes needed. The feature operates synchronously within the REST request and honors existing notification preferences.
- **No action required** for existing sites; the backport to WordPress Core ships this behavior natively.

## Technical details

- Registers `gutenberg_notify_note_mentions()` on `rest_insert_comment` (priority 10, 3 args) to intercept `note` comment insertions.
- Parses mention markup (`<span class="wp-note-mention user-N">@Name</span>`) using `WP_HTML_Tag_Processor` in `gutenberg_get_note_mentioned_user_ids()`, requiring both the `wp-note-mention` and `user-N` classes.
- Filters recipients by excluding the note author, the post author (core handles post-author notifications), and users lacking `edit_comment` capability on the note.
- Sends emails synchronously via `gutenberg_send_note_notification()`, composing content in the recipient's locale with `switch_to_user_locale()` and linking to the post editor via `get_edit_post_link()`.
- Honors the `wp_notes_notify` option and only fires on note creation (`$creating === true`), explicitly skipping updates to prevent duplicate notifications.
- Includes a compatibility shim in `lib/compat/wordpress-7.1/notes-mentions.php` that removes Core's `wp_notify_note_mentions` callback if Gutenberg is active to prevent duplicate emails.
- A targeted `wp_kses_allowed_html` allowance was added to preserve mention markup for users without `unfiltered_html` during REST note saves.

## Contribution

Opened by @adamsilverstein and merged after iterative review, the PR was deliberately scoped to deliver only the notification layer, with the per-thread followers model split out to #80279. Review cycles uncovered several edge cases: the initial permission check used `read_post` (risking content leaks to subscribers), which was corrected to `edit_comment`; mention markup was temporarily stripped by `wp_filter_kses` for non-`unfiltered_html` users, requiring a targeted kses allowance scoped to REST note saves; and a CI failure from a return-type mismatch in `gutenberg_send_note_notification()` was patched. The change was coordinated for backporting to WordPress Core via `wordpress-develop#12548` while remaining in Gutenberg's compat layer.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

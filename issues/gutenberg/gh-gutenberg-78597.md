# #78597: RTC: Fix Edit/Join row action invisible on mobile in post list

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @taipeicoder
- **Labels:** `[Type] Bug`, `First-time Contributor`, `[Feature] Real-time Collaboration`
- **Merged:** [`9280b6c`](https://github.com/WordPress/gutenberg/commit/9280b6c23b013caf46d93ce06e082e0370ea74b2)
- **Discussion:** [#78597](https://github.com/WordPress/gutenberg/pull/78597) · 3 comments · 0 reactions

## Summary

Fixes a regression where the Real-Time Collaboration (RTC) “Edit”/“Join” row action becomes invisible on mobile viewports (≤782px) in `wp-admin/edit.php`. The fix restructures the HTML so the visible label sits directly inside the `<a>` element, bypassing WordPress core’s responsive font-size reset that was collapsing nested spans to 0px.

## Impact

['**Site owners & editors**: RTC row actions now render correctly on mobile devices when Real-Time Collaboration is active.', '**Plugin/theme developers**: No public API changes. Developers overriding the post list table or targeting `.row-actions` should note the updated DOM structure (action links are now direct children of sibling spans rather than nested within a single `<a>`).', '**Hosting & platform teams**: No action required.']

## Technical details

['- **File**: `lib/compat/wordpress-7.1/collaboration.php`', '- **Function**: `gutenberg_post_list_collaboration_row_actions( $actions, $post )`', '- **Root cause**: Core’s mobile CSS applies `.row-actions span { font-size: 0; }` and restores it on `.row-actions span a`, assuming the text node sits directly inside the `<a>`. The previous implementation nested the visible text behind two inner `<span>` layers, triggering the `font-size: 0` rule and collapsing the label.', '- **Fix**: Restructures each state into its own `<span>` wrapper containing a direct-text-leaf `<a>`. The toggle classes (`edit-action-text`, `join-action-text`) move to the outer spans, while descriptive accessible names now use `aria-label` on each link. The heartbeat-driven visibility toggle via `.wp-collaborative-editing`/`.wp-locked` remains untouched.', '- **DOM shift**:', '```html', '<!-- Before -->', '<a href="..."><span class="edit-action-text"><span>Edit</span><span class="screen-reader-text">...</span></span><span class="join-action-text">...</span></a>', '', '<!-- After -->', '<span class="edit-action-text"><a href="..." aria-label="Edit &#8220;Post&#8221;">Edit</a></span>', '<span class="join-action-text"><a href="..." aria-label="Join editing &#8220;Post&#8221;">Join</a></span>', '```']

## Contribution

['Merged by @taipeicoder (first-time contributor) with co-authorship from @shekharnwagh. The PR resolved a CSS specificity conflict discovered during mobile viewport testing by flattening the DOM structure to match core’s responsive baseline. The approach uses sibling `<a>` elements to preserve CSS-only visibility toggling while satisfying `font-size` inheritance rules. AI-assisted tooling was disclosed per project guidelines.']

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

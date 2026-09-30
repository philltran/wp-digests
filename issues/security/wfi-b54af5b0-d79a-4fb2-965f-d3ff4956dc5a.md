# WordPress Core <= 7.1 - Authenticated (Author+) Information Exposure via attachment_submitbox_metadata()

- **Source:** Wordfence Intelligence
- **Type:** Vulnerability
- **Published:** 2026-09-17
- **CVSS:** 4.3 (Medium)
- **CWE:** CWE-200: Exposure of Sensitive Information to an Unauthorized Actor
- **Affected:** WordPress (core) <= 6.6.7; >= 6.7 and <= 6.7.7; >= 6.8 and <= 6.8.8; >= 6.9 and <= 6.9.7; >= 7.0 and <= 7.0.4; >= 7.1 and <= 7.1
- **Patched in:** 6.6.8, 6.7.8, 6.8.9, 6.9.8, 7.0.5, 7.1.1
- **Researchers:** HDWSec
- **Link:** [https://www.wordfence.com/threat-intel/vulnerabilities/id/b54af5b0-d79a-4fb2-965f-d3ff4956dc5a](https://www.wordfence.com/threat-intel/vulnerabilities/id/b54af5b0-d79a-4fb2-965f-d3ff4956dc5a)
- **Usefulness:** 5/5

## Summary

WordPress core's `attachment_submitbox_metadata()` did not check `read_post` against an attachment's parent post before exposing information about it. As a result, an authenticated user with `upload_files` (Author role and above by default) could view the **title** of a private or otherwise unreadable post that the attachment is attached to. The fix adds the missing capability check on the parent post. Patched releases are available on every affected branch.

## Impact

- **Site owners / hosting & platform:** Update core to the patched release for your branch. Sites with auto-updates for minor or security releases enabled should receive it automatically. Confirm that pinned or managed installs are not held on an affected version.
- **Multi-author and editorial sites:** The exposure is limited to parent-post **titles**. The record does not describe any exposure of post content. It matters most where private or draft post titles are themselves sensitive, such as unannounced launches or embargoed stories, and where users hold `upload_files` but not `read_post` on those posts.
- **Plugin & theme developers:** No API change is described. If you render parent-post details for attachments in your own admin UI (for example, custom media screens or meta boxes), apply the same `current_user_can( 'read_post', $parent_id )` guard.
- **Headless & REST consumers:** The advisory names only `attachment_submitbox_metadata()`. No REST route is identified.

## Technical details

- **Vulnerable function:** `attachment_submitbox_metadata()`.
- **Root cause:** the function output the title of the attachment's parent post (`post_parent`) without verifying that the current user can read that post.
- **Class:** information exposure. Exploitation requires an authenticated user with the `upload_files` capability.
- **Fix:** add a `read_post` capability check on the parent post before its title is disclosed.

The advisory does not include the patch diff. The pattern the fix implies is:

```php
// Before: parent title shown unconditionally
$parent = get_post( $post->post_parent );
// ... output get_the_title( $parent ) ...

// After: gated on read access to the parent
if ( $post->post_parent && current_user_can( 'read_post', $post->post_parent ) ) {
    // ... output parent title ...
}
```

Treat the snippet as illustrative. The advisory does not show the exact patched code.

## Contribution

The advisory record contains no discussion or development detail beyond credits and version identifiers.

---

*This record contains material that is subject to copyright. Copyright 2012-2026 Defiant Inc. Defiant hereby grants you a perpetual, worldwide, non-exclusive, no-charge, royalty-free, irrevocable copyright license to reproduce, prepare derivative works of, publicly display, publicly perform, sublicense, and distribute this software vulnerability information. Any copy of the software vulnerability information you make for such purposes is authorized provided that you include a hyperlink to this vulnerability record and reproduce Defiant's copyright designation and this license in any such copy. Licence terms: https://www.wordfence.com/wordfence-intelligence-terms-and-conditions/*

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

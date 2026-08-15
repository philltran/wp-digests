# Accessibility Improvements in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Joe Dolson
- **Published:** 2026-08-13
- **Tags:** `General`, `7.1`, `accessibility`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/08/13/accessibility-improvements-in-wordpress-7-1/](https://make.wordpress.org/core/2026/08/13/accessibility-improvements-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1's accessibility work lands 45 fixes and enhancements in core plus 43 in the editor, with three items that carry developer-visible consequences. Core gains a standardized mechanism for accessible tooltips (#51006), already applied to metabox order buttons (#50921) and the login "Remember Me" control (#55343). List tables now treat the post title cell as the row header instead of the selection checkbox (#32892, #65743), and KSES learns to allow invoker command attributes (#64576) and the `autofocus` attribute on `dialog` elements (#65491). The release also ships a documented, known accessibility regression: the media library switches from a "Load more" button to infinite scroll by default, with an opt-out per user and via the existing `media_library_infinite_scrolling` filter. The block editor adds new Tabs (#80163) and Playlist (#80203) blocks and a decorative-image toggle on the Image block (#78064).

## Impact

**Site owners / admins**
- The media library grid switches to **infinite scroll by default** — a known inaccessible pattern. Each user can turn it off from their User Profile screen. Sites with accessibility obligations should plan for this before upgrading.
- Admin color schemes get contrast increases for sidebar/block-editor compatibility (#65382), and focus indicators are standardized to at least 2 CSS pixels (#65645) — expect visual differences in the admin.

**Plugin & theme developers**
- To disable media library infinite scroll site-wide, return `false` from the existing `media_library_infinite_scrolling` filter. (The WP Accessibility plugin ships a feature that inverts the default and flips the profile option to opt-*in*.)
- KSES now permits invoker command attributes (#64576) and `autofocus` on `dialog` (#65491), so markup previously stripped from post/admin content can survive filtering.
- List table markup changed: the post title cell, not the checkbox cell, is now the row header. Selectors and screen-reader expectations built around the old structure should be re-checked (#32892, #65743). Untitled posts may render an excerpt in place of an empty title (#65022).
- Accordion blocks **removed** arrow/Home/End key navigation (#75891) due to conflicting behaviors — if your product documented or depended on those shortcuts, they are gone.
- `@wordpress/components` consumers: `RadioControl` fieldsets now use the `radiogroup` role (#76745), help text associations changed for `ComboboxControl` (#76761) and `ToggleGroupControl` (#76740), `Dialog` now focuses content rather than the close button (#76910), and the `Page` component header defaults to `h1` (#77617) — that last one can alter heading outlines in custom admin screens.
- Editor code should use `speak()` rather than locally-scoped `aria-live` regions for announcements (#79600).
- Cover block gains attributes to restrict video providers (#80092); Image block gains an explicit decorative toggle (#78064).

**Hosting & platform teams**
- Route-based admin pages now render feedback when JavaScript is unavailable or fails (#65690), which changes what a broken-JS admin request looks like in support triage.
- No action required beyond validating the media library default and any admin CSS that overrides focus styles.

## Technical details

**Tooltips.** Core adds a shared mechanism for exposing accessible names via tooltip (#51006). Its first consumers are the metabox order buttons (#50921) and the login form's "Remember Me" option (#55343). The editor side gained parallel work: `DataForm` panel edit buttons now expose accessible names through tooltips (#77024), and block movement tooltips display their keyboard shortcuts (#76992).

**KSES.** Two attribute families are now allowed through: invoker command attributes (#64576) and `autofocus` on `dialog` (#65491). Markup using the invoker commands API or an autofocusing dialog is no longer stripped.

**List tables.** The row header moves from the selection checkbox cell to the post title cell (#32892, #65743). Page lists add structural indicators for subpages (#64932), and post lists can surface an excerpt for posts with no title (#65022).

**Media library infinite scroll.** The default flips from a "Load more" button to infinite scroll. Two escape hatches exist:

```php
// Disable media library infinite scroll site-wide.
add_filter( 'media_library_infinite_scrolling', '__return_false' );
```

Per-user, a User Profile option turns it off. A per-user toggle inside the existing media modal was attempted in #65775 but did not reach consensus in time for 7.1. The Gutenberg experimental media library replacement governs infinite scroll with an in-modal toggle, currently not enabled and not fully accessibility-reviewed.

**Components and admin screens.** `RadioControl` fieldsets adopt the `radiogroup` role (#76745). `ValidatedRangeControl` no longer accepts invalid object-based labels (#77042). Help text is properly associated in `ComboboxControl` (#76761) and `ToggleGroupControl` (#76740). Focus traps broken by certain `display` values are fixed (#77381); `Dialog` prioritizes focusing content over the close button (#76910); `ContentEditableControl` label associations are corrected (#80344); `Button` disabled-focusable defaults (#78526), forced-colors loading indicators (#78820), and click feedback styling (#76833) are adjusted. The `Page` component header is promoted to `h1` by default (#77617). Focus rings render consistently in High Contrast mode on Tab panels (#77469) and `CollapsibleCard` headers (#77468).

**Visual Revisions.** `title` attributes are replaced with `aria-describedby` (#80440), autosaves get explicit timeline labels (#79950), revision counts are properly pluralized (#78382), CSS outlines act as a non-color secondary indicator for changed documents (#78393), and the revisions slider auto-focuses when the editor is activated (#79691).

**Blocks.** New Tabs (#80163) and Playlist (#80203) blocks ship. Breadcrumbs hides decorative separators from assistive technology (#78524). Cover gains attributes limiting which video providers can be embedded (#80092). Accordion removes arrow/Home/End navigation (#75891).

**Front end.** Password-protected content receives styling and usability changes (#64569), the RSS widget no longer breaks when multiple instances are present (#47670), and the comment reply link stops firing unexpectedly on touch (#46713).

## Contribution

The one live disagreement in the record is #65775 — adding an in-modal toggle for media library infinite scroll to the *existing* media library. That work was in progress but, per the note, "did not reach consensus in time to land in WordPress 7.1," which is why 7.1 ships the infinite scroll default with only a User Profile option and the `media_library_infinite_scrolling` filter as mitigations. Separately, the WP Accessibility plugin added a feature that inverts the core default to `false` and reframes the profile setting as opt-in, effectively shipping the alternative behavior outside core. The dev note was reviewed by @annezazu, @amykamala, and @sabernhardt.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

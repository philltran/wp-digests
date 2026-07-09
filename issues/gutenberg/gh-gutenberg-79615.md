# #79615: Responsive style states: Update responsive editing help text and avoid showing desktop badge

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @talldan
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block editor`, `[Feature] Style States`
- **Merged:** [`c720577`](https://github.com/WordPress/gutenberg/commit/c720577907243a6303fa59269dba31fa59372811)
- **Discussion:** [#79615](https://github.com/WordPress/gutenberg/pull/79615) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request refines the Block Editor’s responsive editing UX by updating copy to clarify that style edits are viewport-specific, and hiding the redundant “Desktop” badge in the block inspector when desktop is selected. The change improves behavioral clarity for the recently introduced responsive style states feature without altering underlying state management or data flow.

## Impact

- **Block editor users & theme/plugin developers:** No code changes required. UI copy and badge visibility in the device dropdown and block inspector will shift to better reflect how style states scope.
- **Hosting/Platform teams:** No action required. Server-side behavior and REST APIs remain unchanged.
- **Developers extending the block editor:** The `isEditingStyleState` calculation now explicitly checks for viewport or pseudo states rather than broadly matching any selected style state, which may affect custom overrides of the block card's edited-content section rendering.

## Technical details

- In `packages/block-editor/src/components/block-inspector/index.js`, `isEditingStyleState` was refactored from `isBlockStyleStateSelected || isResponsiveEditing` to `(hasViewportBlockStyleState( selectedBlockStyleState ) && isResponsiveEditing) || hasPseudoBlockStyleState( selectedBlockStyleState )`. This prevents the desktop state badge from rendering when responsive editing is active on the base viewport, reducing user confusion about edit scope.
- In `packages/block-editor/src/hooks/states.js`, the `default` (Desktop) entry was removed from `DEVICE_STATE_OPTIONS`. Desktop is now treated as the implicit base state rather than an explicit toggle in the device dropdown.
- In `packages/editor/src/components/preview-dropdown/index.js`, i18n strings for tablet/mobile preview options were updated to “Make tablet/mobile exclusive style changes.” and the responsive editing info text changed from “Edits apply only to the current state.” to “Style changes apply only to the current viewport.”

## Contribution

Opened and merged as PR #79615 by @talldan, with co-authors @t-hamano and @tellthemachines. Reviewers discussed conditionally disabling the “Responsive editing” toggle when on desktop but concluded it would add unnecessary workflow friction and make the feature harder to discover. The team prioritized copy clarity and badge visibility over additional UI branching before landing the changes.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

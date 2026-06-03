# What’s new in Gutenberg 23.2? (21 May)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Paulo Trentin
- **Published:** 2026-05-21
- **Tags:** `General`, `block-editor`, `core-editor`, `gutenberg`, `gutenberg-new`
- **Link:** [https://make.wordpress.org/core/2026/05/21/whats-new-in-gutenberg-23-2-21-may/](https://make.wordpress.org/core/2026/05/21/whats-new-in-gutenberg-23-2-21-may/)

## Summary

Gutenberg 23.2 introduces responsive global block styles with states, enabling editors to define breakpoint-specific styling directly within the Global Styles interface. The editor's design system ships new motion design tokens (duration and easing) in `@wordpress/theme`, while the `Modal` component now adapts to narrow viewports by rendering as a bottom sheet on mobile. Additional releases include context-aware shortcode-to-block transforms, Grid component drag-handle refinements, and critical real-time collaboration data layer fixes.

## Impact

- **Theme & Site Builders**: Customizing block styles per viewport is now native. A new "States" dropdown in Global Styles → Blocks allows styling overrides for Tablet and Mobile breakpoints.
- **Component Consumers**: `@wordpress/theme` exposes `duration` and `easing curve` tokens, adopted by Dialog, Modal, and Menu/Dropdown components to standardize animation timing. The `Modal` component automatically switches to a bottom-sheet layout on narrow viewports. A new `SelectControl` primitive is available in `@wordpress/ui`.
- **Block & Plugin Developers**: The Shortcode block now offers block-specific transforms when pasted text matches a registered shortcode. The Embed block natively recognizes `[ embed ]` shortcodes with tightened raw URL matching logic.
- **No action required** for standard theme/plugin functionality unless you are utilizing the new `@wordpress/ui` primitives or relying on the updated mobile modal layout.

## Technical details

- **Responsive Global Styles with States**: Client-side style states were refactored to use nodes (`#78000`), enabling the new "States" dropdown in Global Styles → Blocks to scope styles to Tablet and Mobile viewports (`#77513`).
- **Motion Design Tokens**: `@wordpress/theme` exposes duration and easing curve tokens, which are now adopted by the Dialog, Modal, and Menu/Dropdown components to standardize animation timing (`#76097`).
- **Modal Bottom Sheet on Mobile**: The `Modal` component detects narrow viewports and renders as a bottom sheet, improving one-handed reachability for touch interactions (`#77956`).
- **Grid Component Enhancements**: The `@wordpress/grid` package introduces a `renderResizeHandle` render prop (`#77811`), an edit-mode overlay for `DashboardGrid` and `DashboardLanes` (`#78199`), document cursor locking during resize gestures (`#77813`), and escape-velocity prevention near viewport edges to avoid resize runaways (`#77854`).
- **Shortcode-to-Block Transforms**: The Shortcode block registers block-specific transforms when pasted content matches a registered shortcode pattern, and the Embed block enforces stricter `isMatch` logic for raw URLs while natively supporting `[ embed ]` shortcodes (`#77937`, `#77944`, `#78021`).

## Contribution

Merged into the Gutenberg 23.2 release on May 21, 2026. The milestone was driven by a high volume of PRs across Block Editor components, real-time collaboration (RTC) data layer fixes, and experimental Content Types management screens. Key contributors include @aduth, @jorgefilipecosta, @ntsekouras, @dmsnell, and @youknowriad, with numerous first-time contributors addressing RTC race conditions and flaky E2E tests. The release consolidated several backend utilities and standardized UI interactions without introducing major architectural overhauls.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

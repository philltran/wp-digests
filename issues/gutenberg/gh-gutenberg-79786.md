# #79786: Fix and permit unitless zeros used in CSS `calc` functions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Bug`, `[Package] Block library`, `[Package] Block editor`, `[Package] Edit Post`, `[Package] Edit Site`, `[Package] UI`
- **Merged:** [`e412445`](https://github.com/WordPress/gutenberg/commit/e41244579be0e5d391c74c4e3e0e9d5b7d8160ef)
- **Discussion:** [#79786](https://github.com/WordPress/gutenberg/pull/79786) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Gutenberg package fixed a CSS specification violation where unitless `0` fallbacks caused `calc()` expressions to incorrectly evaluate to zero in modern browsers. The change updates the shared `@wordpress/stylelint-config` to explicitly permit unitless zeros inside custom properties and `var()` functions, and audits internal editor styles to replace invalid unitless values with explicit `0px`. This prevents silent layout breakage when the block editor uses viewport-relative calculations in fullscreen or distraction-free modes.

## Impact

["- **Block Editor & Site Editor developers:** Layouts relying on or overriding variables like `--wp-admin--admin-bar--height` inside `calc()` will now compute correctly. No migration required, as the fix is internal to the editor's CSS layer.", '- **Plugin/Theme Developers:** No action required. The lint rule adjustment applies to `@wordpress/stylelint-config`, not to frontend themes or plugins unless they explicitly extend it.', '- **Editor UI & Style Config:** Developers maintaining internal package styles should note that inline `stylelint-disable length-zero-no-unit` comments are now obsolete and can be removed.']

## Technical details

The core change lives in `packages/stylelint-config/index.js`, which expands the configuration for the `length-zero-no-unit` rule:
```js
// Before
'length-zero-no-unit': true,
// After
'length-zero-no-unit': [
  true, 
  { ignore: ['custom-properties'], ignoreFunctions: ['var'] }
],
```
The unified diff applies this new allowance while auditing and fixing several SCSS/CSS modules where unitless zeros were passed to `calc()` or assigned to variables. For example, in `packages/block-editor/src/components/iframe/content.scss`:
```css
/* Before */
$scroll-top: var(--wp-block-editor-iframe-zoom-out-scroll-top, 0);
/* After */
$scroll-top: var(--wp-block-editor-iframe-zoom-out-scroll-top, 0px);
```
Similar adjustments were made to `packages/boot/src/components/root/style.scss`, `packages/edit-post/src/style.scss`, `packages/edit-site/src/components/layout/style.scss`, and `@wordpress/ui` modules. All legacy inline `/* stylelint-disable length-zero-no-unit */` overrides were stripped once the config natively permitted the pattern. Test coverage was appended to `packages/stylelint-config/test/values-valid.css` to prevent regression.

## Contribution

Opened by @aduth to address a CSS Values-4 spec violation where unitless zeros break math functions during addition/subtraction in browser renderers. During review, @ciapo suggested adding explicit test coverage for valid zero-length variables, which was committed in `22c10ed`. The PR was merged as `e412445`, with co-author credit noted from @ciapo. No competing design approaches were evaluated; the solution focused on aligning the lint configuration with browser spec behavior while cleaning up legacy inline disables.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

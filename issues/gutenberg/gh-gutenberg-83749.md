# #83749: Base Styles: Default the admin theme color to Blueberry

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamwoodnz
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Block editor`, `First-time Contributor`, `[Package] Base styles`
- **Merged:** [`f23a2ce`](https://github.com/WordPress/gutenberg/commit/f23a2ce1164c9e70412b80ddd5cd8d1fab64bfb9)
- **Discussion:** [#83749](https://github.com/WordPress/gutenberg/pull/83749) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `:root` default for `--wp-admin-theme-color` in `@wordpress/base-styles` changes from the legacy `#007cba` to `#3858e9`, the WordPress 7.0 Modern (Blueberry) admin scheme color. `@wordpress/components` already defaulted to `#3858e9`. Before this change, any shared base-styles stylesheet that loaded after `components` reset the accent to `#007cba` outside wp-admin, for example in Storybook or standalone apps. To keep the Fresh scheme on `#007cba`, the PR adds explicit `body.admin-color-fresh` rules.

## Impact

- **Plugin and theme developers who compile base-styles defaults** (for example WooCommerce-style builds that bundle `_default-custom-properties.scss`):
  - Your `:root` accent is now `#3858e9` unless a scheme class overrides it.
  - To keep the old color, include `admin-scheme(#007cba)` on `:root` in your own stylesheet. The changelog documents this.
- **Standalone apps, Storybook and other non-wp-admin consumers:** the default accent becomes blueberry. Overrides such as Jetpack's Storybook workaround (Automattic/jetpack#52849) should no longer be needed.
- **Site owners:**
  - Modern users see `#3858e9`.
  - Fresh users stay on `#007cba`, because the PR adds a Fresh rule in both the mixin and the defaults.
  - Other schemes such as Ocean are unchanged.
- **Mixed-version setups:** older WordPress `admin-schemes.css` has no Fresh rule. The same Fresh rule is therefore shipped alongside the `:root` defaults, so Fresh stays `#007cba` there too.
- No API removals or renames.

## Technical details

Changes in `packages/base-styles`:

- `_default-custom-properties.scss`: the `:root` block now calls `mixins.admin-scheme(#3858e9)` instead of `#007cba`. A new `body.admin-color-fresh { @include mixins.admin-scheme(#007cba); }` block follows it.
- `_mixins.scss`:
  - `wordpress-admin-schemes()` gains a `body.admin-color-fresh` rule using `admin-scheme(#007cba)`. The `light` scheme keeps `#007cba`.
  - The `link-reset` focus fallback changes to `var(--wp-admin-theme-color, #3858e9)`.
- The `var(--wp-admin-theme-color, #007cba)` fallbacks become `#3858e9` in:
  - `block-editor` global-styles `inheritance/style.scss`
  - `editor` Style Book `constants.ts` (`STYLE_BOOK_IFRAME_STYLES`)
  - `media-editor` `cropper.scss`
- `CHANGELOG.md` gets an Enhancements entry.

```scss
// before
:root { @include mixins.admin-scheme(#007cba); }

// after
:root { @include mixins.admin-scheme(#3858e9); }
body.admin-color-fresh { @include mixins.admin-scheme(#007cba); }
```

`admin-scheme()` sets both `--wp-admin-theme-color` and its `--rgb` variant, so the compiled `common.css` has `#3858e9` and `56, 88, 233`. There are no DB, REST or block.json changes.

## Contribution

This is a first-time contributor PR by @adamwoodnz that closes #83748. It follows the WordPress 7.0 switch of the default admin color scheme to Modern (Trac #64546) and relies on the earlier `components` default in #50193. The description discloses that it was authored with Claude Code. The PR thread shows no design debate. The bot credits @mirka and @jasmussen as interacting accounts. The PR's main design choice was to add the explicit Fresh rule in two places so Fresh users don't regress, including on older WordPress versions.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

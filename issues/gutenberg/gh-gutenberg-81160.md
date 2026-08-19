# #81160: Global styles: Register <label> as element

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @sarthaknagoshe2002
- **Labels:** `[Type] Enhancement`, `[Package] Blocks`, `[Package] Block library`
- **Merged:** [`a81f047`](https://github.com/WordPress/gutenberg/commit/a81f0474d9c189a559af45ac64cb10bfe9264f7c)
- **Discussion:** [#81160](https://github.com/WordPress/gutenberg/pull/81160) · 9 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This registers `label` as a global-styles element, so `theme.json` can now carry a `styles.elements.label` block that applies to every `<label>` tag on the front end and in the editor — not just labels rendered by core blocks. It continues the recent form-element work in global styles (which already covers `textInput` and `select`), letting a theme style form labels consistently regardless of which blocks or third-party plugins emit the form. To keep the new element styles from being outranked by block defaults, the PR also rewrites the label CSS in Categories, Comments, Post Comments Form, Form Input, and Search to use `:where()` so those defaults sit at zero specificity.

## Impact

**Theme developers**
- New `styles.elements.label` key in `theme.json`, accepting the full style property set (`color`, `typography`, `spacing`, `border`, etc.) — the same breadth as other complete elements, not a restricted subset.
- The selector is the bare `label` tag, so a single declaration reaches core form blocks (Archives, Categories, Form, Form Input, Post Comments Form, Search) *and* any plugin- or Custom-HTML-rendered `<label>`. Themes that only wanted to style core block labels should scope with block-level styles instead.
- Global styles UI/`theme.json` validation will accept the key; the bundled JSON schema (`schemas/json/theme.json`) was updated so editors give autocomplete and stop flagging it.

**Plugin developers**
- Any `<label>` your blocks or forms output can now be restyled by the active theme without you doing anything. If your markup must not inherit theme label styles, you need selectors that outrank a bare-element global style.

**Site owners / theme users**
- Default label styling for the affected core blocks (e.g. Search's bold label, the block-level `display: block; margin-bottom: 0.25em` on comment-form labels) is now inside `:where()`. Any custom CSS or theme rule that previously lost the specificity fight with those defaults may now win, so front-end appearance can shift on sites carrying such overrides. Worth a visual check on comment forms and search blocks.

**No action required** for anyone not styling labels — the change adds an opt-in key and does not alter default rendering when `theme.json` says nothing about `label`.

## Technical details

Three registration points plus a CSS-specificity pass.

- `lib/class-wp-theme-json-gutenberg.php` — adds `'label' => 'label'` to the elements map, alongside `caption`, `cite`, `select`, and `textInput`. This is what generates the `label { … }` rule from `styles.elements.label`.
- `packages/blocks/src/api/constants.ts` — adds `label: 'label'` to `__EXPERIMENTAL_ELEMENTS`, the JS-side mirror used by the editor so styles render in the canvas as well as on the front end.
- `schemas/json/theme.json` — adds `label` under the elements definition as `{ "$ref": "#/definitions/stylesPropertiesComplete" }`. Note this differs from neighbours like `select` and `cite`, which compose with `allOf` to narrow the allowed properties; `label` gets the complete property set.

Usage:

```json
"styles": {
  "elements": {
    "label": {
      "color": { "text": "red", "background": "blue" },
      "spacing": {
        "padding": { "top": "25px", "right": "25px", "bottom": "25px", "left": "25px" }
      }
    }
  }
}
```

The specificity work in `packages/block-library/src/*/style.scss` (and `search/theme.scss`) moves block label defaults out from under their block wrappers and into zero-specificity `:where()` rules, so an element-level `label { … }` declaration can override them. For example, Search:

```scss
// Before
.wp-block-search {
  .wp-block-search__label { font-weight: bold; }
}

// After
:where(.wp-block-search .wp-block-search__label) { font-weight: bold; }
```

The same treatment is applied to `.wp-block-categories__label:not(.screen-reader-text)`, `.wp-block-form-input__label` / `__label-content`, and the `comment-form-comment|author|email|url label` rules in both `comments` and `post-comments-form`. Those comment-form rules are additionally hoisted out of the block's nesting and re-anchored on `.wp-block-post-comments` / `.wp-block-post-comments-form` to preserve scope at the lower specificity. No PHP behavior, REST schema, or database change beyond the theme.json shape.

## Contribution

Part of the long-running form-elements effort in issue #34198 and closing #81647. The substantive review question came from @MaggieCabrera, who asked whether existing core blocks (Search, Comments) should "opt in" to the new element; @scruffian read that as declaring the label rules in `block.json` instead. The author's counter — that targeting the raw `<label>` tag means those blocks inherit automatically, provided no higher-specificity block CSS wins — was accepted, and the ask reframed into what actually shipped: audit each block's default CSS, lower its specificity, and prove it with side-by-side QA. @MaggieCabrera pointed at PR #70378 as the precedent for that pattern, and the author followed it, adding a three-way Trunk / PR-default / PR-styled screenshot comparison. A backport changelog entry for 7.1 was added pointing at wordpress-develop PR #13105.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

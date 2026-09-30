# #79584: Add textShadow typography support and UI

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Enhancement`, `[Feature] Block API`, `[Package] Blocks`, `[Package] Block library`, `[Package] Block editor`, `Needs Dev Note`, `[Package] Style Engine`
- **Merged:** [`f87059c`](https://github.com/WordPress/gutenberg/commit/f87059c83f0278e1944c2a80678d577bf487f4d3)
- **Discussion:** [#79584](https://github.com/WordPress/gutenberg/pull/79584) · 36 comments · 1 reactions
- **Usefulness:** 5/5

## Summary

Adds full `textShadow` support to Gutenberg: a new `supports.typography.textShadow` block support, `settings.typography.textShadow` / `defaultTextShadowPresets` / `textShadowPresets` in `theme.json`, and editor UI in both the block inspector and Global Styles. Presets emit `--wp--preset--text-shadow--{slug}` custom properties and `.has-{slug}-text-shadow` classes. Paragraph and Heading opt in, and the core `lib/theme.json` ships two default presets (`light`, `strong`). It builds on the earlier style-only support from #73320.

## Impact

**Theme developers**
- New `theme.json` settings under `settings.typography`: `textShadow` (default `true`), `defaultTextShadowPresets` (default `true`), and `textShadowPresets` (`[ { name, slug, textShadow } ]`). To remove the core presets, set `defaultTextShadowPresets: false` or `textShadowPresets: []`; to disable the feature entirely, set `textShadow: false`.
- `styles.typography.textShadow` (global, per-block, per-element) is accepted, e.g. `styles.blocks['core/paragraph'].typography.textShadow`.
- Core presets are fixed-colour black shadows. As noted in review, they can be hard to see on dark style variations (the same limitation already applies to colour and shadow presets). Dark/light adaptation was deferred as a separate issue.
- The `Needs Dev Note` label is set, so official guidance is expected.

**Plugin / block developers**
- New opt-in: `supports.typography.textShadow: true`. The control is hidden by default unless the theme declares support.
- Adds a `textShadow` string attribute (preset slug) and uses the shared `style` attribute for custom values. Serialization skipping is supported via `__experimentalSkipSerialization`, though no core block uses it yet.
- Preset selections are saved as a CSS class rather than inline style: `<!-- wp:paragraph {"textShadow":"strong"} --><p class="has-strong-text-shadow">`.

**Site owners**
- Once shipped, Paragraph and Heading blocks gain a text-shadow control, and Global Styles gains a text-shadow presets screen next to font sizes.

**Core / release note**
- The backport changelog targets WordPress 7.2 (`backport-changelog/7.2/13035.md`). Per the discussion, the PR was at risk of missing 7.1 because of the open question over default presets.

## Technical details

Changes visible in the (truncated) diff:

- **`lib/block-supports/typography.php`**: `gutenberg_register_typography_support()` reads `$typography_supports['textShadow']`, includes it in `$has_typography_support`, and registers a `textShadow` string attribute if absent. `gutenberg_apply_typography_support()` honors `wp_should_skip_block_supports_serialization( ..., 'textShadow' )` and builds `$typography_block_styles['textShadow']` from either the preset attribute (`var:preset|text-shadow|{slug}`) or `style.typography.textShadow`, with the preset taking precedence. Output goes through `gutenberg_style_engine_get_styles()`.
- **`lib/class-wp-theme-json-gutenberg.php`**: new `PRESETS_METADATA` entry for `typography.textShadowPresets` (`prevent_override` => `typography.defaultTextShadowPresets`, `value_key` => `textShadow`, `css_vars` => `--wp--preset--text-shadow--$slug`, class `.has-$slug-text-shadow` => `text-shadow`). `VALID_SETTINGS['typography']` gains `textShadow`, `defaultTextShadowPresets`, `textShadowPresets`.
- **`lib/theme.json`** defaults: `textShadow: true`, `defaultTextShadowPresets: true`, and presets `light` (`0.05em 0.05em 0.1em rgba(0, 0, 0, 0.3)`) and `strong` (`0.1em 0.1em 0.25em rgba(0, 0, 0, 0.5)`). `lib/theme-i18n.json` adds translatable `name` for `textShadowPresets` (top level and per-block).
- **`lib/compat/wordpress-7.2/kses.php`** (new, loaded from `lib/load.php`): a `safe_style_css` filter adds `text-shadow` so it survives KSES.
- **Block editor**: `useSettingsForBlockElement()` in `global-styles/hooks.js` adds `textShadow` to the list of keys checked against a block's supported styles; new styles in `global-styles/style.scss` for the text-shadow picker (`.block-editor-global-styles__text-shadow*`). The rest of the UI code is in the truncated portion of the diff and is not described here.
- **Docs**: `block-supports.md`, `theme-json-living.md`, `global-settings-and-styles.md`, and the core blocks README (Paragraph and Heading now list `textShadow`).

Example block registration:

```json
{
  "supports": {
    "typography": { "textShadow": true }
  }
}
```

## Contribution

The author (@t-hamano) flagged early that the volume of code and need for design review made 7.1 uncertain, with a fallback of shipping only the style support from #73320 in 7.1. The main blocker became choosing the default preset(s), since they can't be changed later. @t-hamano suggested deferring to 7.2 for broader design feedback; @jasmussen agreed and proposed following the drop-shadow precedent, reusing its naming and style (e.g. porting "Natural" and "Deep" with reduced depth and blur, skipping ones like "Outlined" and "Crisp" that would hurt legibility). @bph's Playground smoke test showed presets not visibly applying on TT5's darker variations, which was treated as a broader preset-contrast issue for a separate discussion. The merged backport changelog targets 7.2.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

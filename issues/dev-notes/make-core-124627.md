# What’s new in Gutenberg 23.6? (July 22, 2026)

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Aki Hamano
- **Published:** 2026-07-22
- **Tags:** `General`, `block-editor`, `core-editor`, `gutenberg`, `gutenberg-new`
- **Link:** [https://make.wordpress.org/core/2026/07/22/whats-new-in-gutenberg-23-6-july-22-2026/](https://make.wordpress.org/core/2026/07/22/whats-new-in-gutenberg-23-6-july-22-2026/)
- **Usefulness:** 4/5

## Summary

Gutenberg 23.6 stabilizes the Playlist and Tabs blocks, introduces a dynamic variation for the Gallery block, and adds a custom icon registration API with collection support. The release also brings inline notes and @mention autocomplete to the Notes system, allows theme authors to customize editor viewport breakpoints via theme.json, and hard-deprecates the legacy __next40pxDefaultSize prop across dozens of UI components. These changes shift several experimental features into production and modernize the editor’s theming and collaboration workflows.

## Impact

- **Plugin & theme developers:** The `__next40pxDefaultSize` prop is hard-deprecated across `BorderBoxControl`, `ComboboxControl`, `FontSizePicker`, `RangeControl`, and other components; migrate to explicit `size` props or the new design system tokens. `@wordpress/reusable-blocks` public APIs are deprecated and should be replaced with core block registration patterns.
- **Block authors:** The Playlist and Tabs blocks are now stable and available without experiment flags. The Gallery block gains a `dynamic` variation that renders all attached media. The Custom HTML block now supports `innerContent` for static inner blocks.
- **Site owners & editors:** The admin bar is now visible by default in both the Post and Site Editors. Viewport breakpoints for tablet and mobile previews can be configured in `theme.json`. "Apply Globally" now lets you select specific style properties instead of applying all at once.
- **No action required** for most users, though theme authors should audit component usage for the deprecated sizing prop.

## Technical details

- **Icon registration:** New PHP methods render inline SVG icons from the registry, and a REST endpoint exposes icon collections. The icon picker now filters by collection and loads icons asynchronously.
- **Notes system:** Inline notes use a hybrid marker + strip-on-render approach to anchor highlights to text selections. `@mention` autocomplete inserts a mention chip carrying the user’s ID and links to their author page.
- **theme.json viewport config:** Tablet and mobile preview widths are now read from `theme.json`, allowing device previews to match theme breakpoints.
- **Gallery dynamic variation:** The `core/gallery` block now includes a `dynamic` variation that queries and displays all post-attached media in both editor and frontend.
- **Component deprecations:** Dozens of components (`BorderBoxControl`, `ComboboxControl`, `CustomSelectControl`, `FontSizePicker`, `FormFileUpload`, `FormTokenField`, `InputControl`, `NumberControl`, `Radio`, `RangeControl`, `SelectControl`, `ToggleGroupControl`, `TreeSelect`, `UnitControl`, `FontAppearanceControl`, `FontFamilyControl`, `LetterSpacingControl`, `LineHeightControl`) hard-deprecate the `__next40pxDefaultSize` prop. Developers should remove it and rely on explicit sizing or CSS variables.
- **Editor architecture:** The Post Editor is now always iframed, aligning its rendering context with the Site Editor. The `@wordpress/reusable-blocks` package public APIs are deprecated in favor of core block registration patterns.
- **Before/after for deprecated prop:**
  ```diff
  - <BorderBoxControl __next40pxDefaultSize />
  + <BorderBoxControl size="medium" />
  ```

## Contribution

This release was driven by a coordinated effort to stabilize long-running experimental blocks and modernize the editor’s design system. Key contributors including @t-hamano, @aduth, @desrosj, and @jasmussen shepherded the Playlist and Tabs blocks through stabilization, while @ellatrix and @sirreal led the icon collection API and @mention autocomplete implementations. The hard deprecation of __next40pxDefaultSize followed months of component migration work, with the team systematically replacing the legacy sizing pattern across the wordpress/components and wordpress/ui packages. The Post Editor’s shift to always-iframed rendering was adopted after extensive testing to align with the Site Editor’s isolation model.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# Editor components updates in WordPress 7.1

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Lena Morita
- **Published:** 2026-07-23
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/07/23/editor-components-updates-in-wordpress-7-1/](https://make.wordpress.org/core/2026/07/23/editor-components-updates-in-wordpress-7-1/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 finalizes the 40px default height rollout for form controls in `@wordpress/components`, removes the deprecated `Navigation` component and `__experimentalApplyValueToSides` utility, and shifts internal styling from Emotion to SCSS modules. These changes eliminate legacy opt-in props, enforce consistent spacing, and simplify the component library's styling architecture. Plugin and theme developers must update their component usage to avoid relying on removed APIs or deprecated styling patterns.

## Impact

- **Plugin & theme developers:** Remove `__next40pxDefaultSize` from all component usage; passing it or `size="__unstable-large"` now has no effect. Replace `Navigation` with `Navigator`. Update Emotion-based styling in `View`, `Divider`, `Surface`, `Truncate`, `Flex`, and `Spacer` to use `style`/`className` and compose `css()` fragments before passing to `cx()`.
- **Hosting & platform teams:** No action required; these are library updates shipped with core.
- **Headless & REST consumers:** No direct impact; changes are confined to the block editor UI layer.

## Technical details

The `@wordpress/components` package unconditionally sets form control heights to `40px`, stripping the runtime logic for `__next40pxDefaultSize` and deprecating the `size` prop on `BorderBoxControl`, `BorderControl`, `FontSizePicker`, and `ToggleGroupControl`. The `Navigation` component and `__experimentalApplyValueToSides` utility are fully removed from the package exports. Internally, the `View` component no longer renders through Emotion, making the legacy `css` prop a no-op and changing how `cx()` resolves cascade order. Developers previously using separate `css()` fragments with `cx()` must now merge them into a single call to preserve override precedence:

```js
// Before (fragments passed separately)
cx(css(baseStyles), css(overrideStyles), className)
// After (merged before cx)
cx(css(baseStyles, overrideStyles), className)
```

Affected components include `BoxControl`, `ComboboxControl`, `CustomSelectControl`, `FormFileUpload`, `FormTokenField`, `FocalPointPicker`, `InputControl`, `NumberControl`, `QueryControls`, `Radio`, `RangeControl`, `SearchControl`, `SelectControl`, `TextControl`, `TreeSelect`, `UnitControl`, and block-editor controls like `FontAppearanceControl` and `FontFamilyControl`.

## Contribution

The changes were coordinated across multiple tracking issues and finalized for the 7.1 cycle. Co-authored by @0mirka00 and @mciampini, with review by @aduth, the work consolidated a long-running Emotion-to-SCSS migration and completed the deprecation lifecycle for components introduced in 6.7 and 6.8. The team deliberately kept the `Button` component on the opt-in path to avoid breaking existing layouts, while fully removing the navigation and utility APIs that had been soft-deprecated for a full release cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

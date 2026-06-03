# #78501: Font Library: clarify active variant state in Library tab

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dhruvikpatel18
- **Labels:** `[Type] Enhancement`, `[Feature] Font Library`
- **Merged:** [`4f7082a`](https://github.com/WordPress/gutenberg/commit/4f7082a522918a0551c1b1750d10683279b082d9)
- **Discussion:** [#78501](https://github.com/WordPress/gutenberg/pull/78501) · 3 comments · 0 reactions

## Summary

The Font Library modal in the Site Editor now displays variant status as “x of y active” instead of “x/y variants active”. This localized string update improves scannability and makes active/inactive state indicators clearer at a glance for users managing typography across theme and custom font sets. No functional or behavioral changes are introduced.

## Impact

- **Theme & Plugin Developers:** No action required. The change affects an internal UI label within the Font Library modal and does not modify public APIs, block attributes, or style generation routines.
- **Site Editors / Content Creators:** Slightly improved scanability when reviewing active font variants in `Styles > Typography > Font Library`. No configuration, migration, or code changes are needed.

## Technical details

The merged diff updates a single localization string in `packages/global-styles-ui/src/font-library/installed-fonts.tsx` within the `InstalledFonts` component.
```tsx
// Before
__( '%1$d/%2$d variants active' )

// After
__( '%1$d of %2$d active' )
```
The change only replaces the sprintf format string passed to the UI renderer. No PHP hooks, REST schema changes, or `block.json` fields were modified. While discussion proposed a color contrast increase and layout adjustments (narrowing list width or repositioning status text), those elements were not included in the final diff.

## Contribution

Merged as commit `4f7082a` by @dhruvikpatel18 with co-authors @im3dabasia, @jameskoster, @jasmussen, @colorful-tones, @karmatosed, and @mikachan. The update closes issue #58276 to address skimmability of variant counts in the Font Library list. During review, @jameskoster suggested layout refinements as a separate future effort, while @jasmussen recommended reverting an unmerged color adjustment to contrast level 700. The merged code exclusively ships the localized string update.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

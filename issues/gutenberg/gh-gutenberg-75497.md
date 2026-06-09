# #75497: Blocks: Allow the `Loginout` block as an inner block in the `Navigation Submenu` block

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @im3dabasia
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Login/out`, `[Block] Submenu`
- **Merged:** [`f60ce2a`](https://github.com/WordPress/gutenberg/commit/f60ce2a16a51373de94cf9ca068a74de3d431985)
- **Discussion:** [#75497](https://github.com/WordPress/gutenberg/pull/75497) · 3 comments · 0 reactions

## Summary

The Login/out block can now be nested as an inner block within the Navigation Submenu block. This resolves a previous editor limitation that prevented developers and site owners from creating common account-management patterns, such as a “My Account” parent item containing a logout link. The change simply updates the submenu block's allowed inner blocks configuration to accept the `core/loginout` namespace.

## Impact

- **Block Editors & Site Owners:** Can now drag and drop the Login/out block into submenu items to build dropdown navigation without manual markup.
- **Theme & Plugin Developers:** No code changes required; existing `wp:navigation-submenu` instances automatically support the new nesting capability upon update.
- **Headless & REST Consumers:** Serialization of the Navigation Submenu block will now include `core/loginout` as a valid inner block, aligning server-side expectations with editor constraints.
- **Action Required:** None. The updated block library version extends the whitelist transparently.

## Technical details

The unified diff modifies `packages/block-library/src/navigation-submenu/edit.js`, appending `'core/loginout'` to the existing `ALLOWED_BLOCKS` array. This array serves as the validation filter for the `wp:navigation-submenu` block's inner block API, dictating which child namespaces are accepted during both runtime editing and serialization. Adding the login/out namespace synchronizes editor-side constraints with the underlying block structure, eliminating the previous error state when attempting to nest the block.

```javascript
// Before
const ALLOWED_BLOCKS = [
    'core/navigation-link',
    'core/navigation-submenu',
    'core/page-list',
];

// After
const ALLOWED_BLOCKS = [
    'core/navigation-link',
    'core/navigation-submenu',
    'core/page-list',
    'core/loginout',
];
```

## Contribution

Opened by @im3dabasia to resolve issue #75408, the PR was identified as a minimal configuration update. @Drivingralle verified the editor behavior, while @dmsnell initiated a cross-review to ensure alignment with existing submenu block constraints. The patch was co-authored by @soean and @drivingralle during review and merged as commit `f60ce2a`. No alternative implementations or architectural debates emerged; the change was processed as a straightforward whitelist extension.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

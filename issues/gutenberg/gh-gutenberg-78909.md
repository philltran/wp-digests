# #78909: PHP-Only blocks: forward current post ID to server render

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @t-hamano
- **Labels:** `[Type] Bug`, `[Package] Block library`
- **Merged:** [`c4ae590`](https://github.com/WordPress/gutenberg/commit/c4ae5900fd6d0f2d391eaa5221b95a41ab7ad32d)
- **Discussion:** [#78909](https://github.com/WordPress/gutenberg/pull/78909) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

PHP-only blocks (server-registered blocks using `supports.autoRegister`, which get an auto-generated `edit` backed by `useServerSideRender`) were previewed in the editor without any post context, so a `render_callback` calling `get_the_ID()` or otherwise relying on the post being edited rendered against the wrong post — or none at all. This change makes the auto-generated `edit` pass the current `postId` block context to the server render request as a `post_id` URL query argument, matching what the older `ServerSideRender` component has always done. It also forces `postId` into the block type's `usesContext` so the value is actually available, regardless of what the PHP registration declared. Editor previews of PHP-only blocks now match front-end output for post-dependent render callbacks.

## Impact

- **Plugin developers building PHP-only blocks** — `render_callback`s that depend on the current post (`get_the_ID()`, `get_post_meta()` on the edited post, anything reading the global `$post`) now receive the correct post during editor preview. No code change required; you do **not** need to declare `'uses_context' => array( 'postId' )` for this to work.
- **Blocks that *did* declare `postId` in `uses_context`** — unaffected in behavior, but note the declared context array is now merged with `postId` client-side rather than replaced, so nothing you declared is dropped.
- **Everyone else** — no action required. This is a preview-only fix; front-end rendering is unchanged.
- **Behavioral note:** the `post_id` argument is now sent unconditionally rather than only for blocks that opt in via `uses_context`. In contexts where no `postId` is available (for example template editing without a post), `context?.postId` is `undefined` and no meaningful post id is forwarded.

## Technical details

The change is a single hunk in `packages/block-library/src/index.js`, in the client-side registration path that wires up bootstrapped (server-registered) PHP-only block types.

Two things changed:

1. **`usesContext` is now merged, not inherited as-is.** The registration object builds `usesContext` from the bootstrapped block type's own `usesContext` plus `'postId'`, deduplicated through a `Set`:

   ```js
   usesContext: Array.from(
       new Set( [
           ...( bootstrappedBlockType?.usesContext ?? [] ),
           'postId',
       ] )
   ),
   ```

   This is what makes `context.postId` reachable inside the generated `edit` even when the PHP registration never declared it.

2. **The generated `edit` forwards the id to the render request.** Before and after:

   ```js
   // Before
   edit: function Edit( { attributes } ) {
       const { content, status, error } = useServerSideRender( {
           block: blockName,
           attributes,
       } );

   // After
   edit: function Edit( { attributes, context } ) {
       const { content, status, error } = useServerSideRender( {
           block: blockName,
           attributes,
           urlQueryArgs: { post_id: context?.postId },
       } );
   ```

The `post_id` query arg is what the block-renderer REST endpoint uses to set up post context before invoking the `render_callback` — this brings `useServerSideRender` in line with the `ServerSideRender` component, which already appended the current post id automatically. No PHP, `block.json`, REST schema, or database changes are involved; the whole delta is +52 B in `build/scripts/block-library/index.min.js`.

## Contribution

Opened by **@t-hamano** to close issue #78619. The original implementation was conditional — `post_id` was only forwarded when the block declared `postId` through its PHP `uses_context` — but that was struck out and replaced with unconditional forwarding after discussion on the linked issue concluded it was better to always include the parameter. Props were recorded for **@priethor**, **@Mamaduka**, **@mcsf**, and **@talldan** alongside the author. A "Backport to WP Minor Release" label was added and then removed by the author, who noted the fix lands in the 7.1 release anyway. Per the PR's own disclosure, the change was authored with assistance from Claude Code and reviewed by the contributor.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

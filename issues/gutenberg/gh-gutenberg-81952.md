# #81952: Menu: Close non-modal menus on iframe interaction

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ciampo
- **Labels:** `[Type] Bug`, `[Package] UI`
- **Merged:** [`a946c3e`](https://github.com/WordPress/gutenberg/commit/a946c3eac86484b62b684bcd797f21553ef74c30)
- **Discussion:** [#81952](https://github.com/WordPress/gutenberg/pull/81952) · 8 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/ui` `Menu.Root` now closes a non-modal menu when a pointer interaction starts inside a same-origin iframe, such as the editor canvas. Base UI 1.7.0 only listens for outside presses in the menu's owner document, and pointer events don't cross document boundaries, so the menu stayed open after a canvas click. The fix is a self-contained, temporary hook that bridges the gap until the upstream Base UI issue (mui/base-ui#5410) is resolved.

## Impact

**Plugin & theme developers using `@wordpress/ui` `Menu`**
- Non-modal menus (`modal={ false }`) now dismiss on a pointer press inside a same-origin iframe. The iframe target still receives the original interaction, including click and focus.
- Behavior of modal menus is unchanged.
- The public `actionsRef` prop still works; a test covers that `actionsRef.current.close()` still closes the menu.
- The `onOpenChange` `eventDetails.reason` for these closes is `imperative-action`, not `outside-press`. Consumers that branch on `outside-press` will not see it for iframe dismissals.

**Limitations**
- Cross-origin iframes are not bridged.
- Initially open cross-document roots are not bridged.

**Gutenberg / editor contributors**
- Unblocks adopting `Menu` in the editor (e.g. the Options menu from #81564) with non-modal behavior, since the editor canvas is an iframe.

No action required otherwise.

## Technical details

The diff adds `packages/ui/src/menu/use-iframe-dismissal-bridge.ts` and wires it into `packages/ui/src/menu/root.tsx`.

**`Root` changes**
- Previously it destructured `onOpenChange` out and spread the remaining `rootProps` onto `_Menu.Root`, passing its own `handleOpenChange`.
- Now it calls `useIframeDismissalBridge( { actionsRef, defaultOpen, disabled, modal, onOpenChange: handleOpenChange, open } )` and renders `<_Menu.Root { ...props } { ...iframeDismissalProps } />`. The hook's returned props override the user-supplied ones (`onOpenChange`, `actionsRef`, and what is needed to track the trigger and open state).
- The `open` parameter in `handleOpenChange` was renamed to `nextOpen`.

**Hook behavior (from the visible diff)**
- Tracks open state, controlled via `open` or uncontrolled via `defaultOpen`, plus the trigger element. It falls back to an internal ref when no `actionsRef` is supplied, so it can call the Base UI action ref's close.
- `useCloseOnIframePointerDown` is enabled only while the menu is open. It walks the owner document's `iframe` elements, including nested iframes, and adds a capture-phase `pointerdown` listener on each same-origin `contentDocument`. The listener is capture-phase and non-consuming, so the iframe target still receives the event.
- A per-iframe `load` listener re-resolves `contentDocument` after reloads and moves the listener to the new document.
- A `MutationObserver` on each observed document adds and removes listeners as iframes are added or removed, covering remounts.
- `getIframeDocument` wraps `contentDocument` access in try/catch, returning `null` for cross-origin frames.
- `isInsideCurrentMenu` resolves the popup through the trigger's `aria-controls`, then checks containment or `data-rootownerid`, so presses inside the menu's own portaled popup (including one portaled into an iframe) don't close it.
- Disabled menus do not close (per the `disabled` input and a dedicated test).
- The hook comment documents the upstream issue and the removal path: delete the import, call, and prop spread in `Root`, and pass `handleOpenChange` directly to `_Menu.Root` again.

The diff also adds unit tests in `packages/ui/src/menu/test/index.test.tsx` for the following:
- Close without consuming the iframe click.
- Nested iframes.
- Menu portaled into an iframe.
- Disabled menus.
- Listener reattach on reload and cleanup on close or unmount.
- Iframe remount while open.
- Preservation of the public `actionsRef`.

The diff also adds a `CHANGELOG.md` entry. The diff is truncated, so the tail of the hook (the close call and returned props) is not visible.

## Contribution

Follow-up to #81564, merged by the author (@ciampo) without a formal approval. Reviewer @mirka asked whether this much temporary code was justified versus waiting for an upstream Base UI fix. @ciampo replied that the upstream issue had no maintainer acknowledgment yet, the bug affects several `Menu` usages because the editor is an iframe, and the alternatives (making menus modal, or rendering an invisible overlay) were less desirable. @Mamaduka tested it with #81564 and agreed a similar hotfix would be needed regardless. @ciampo merged because the bug was blocking `Menu` adoption in the editor, leaving further changes for follow-ups.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

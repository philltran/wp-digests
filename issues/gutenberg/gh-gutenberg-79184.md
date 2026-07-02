# #79184: [RTC] Add granular collaboration control

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shekharnwagh
- **Labels:** `[Type] Enhancement`, `[Package] Editor`, `[Package] Edit Post`, `[Feature] Real-time Collaboration`, `[Package] Edit Site`
- **Merged:** [`e37c19d`](https://github.com/WordPress/gutenberg/commit/e37c19deaa7cb3aa67794754531ecef660fa9701)
- **Discussion:** [#79184](https://github.com/WordPress/gutenberg/pull/79184) · 18 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The block editor's real-time collaboration (RTC) notification system replaces its monolithic master toggle with three independent preferences: join, leave, and post-save notifications. This change gives users finer control over collaboration toasts in both the post and site editors and exposes a new JavaScript filter for default customization.

## Impact

- **Plugin & theme developers**: Can now set editor-level default states for RTC toast notifications via the new `editor.CollaborationNotificationPreferenceDefaults` filter.
- **Site editors & agency teams**: Gain granular control in Preferences > General under a new "Collaboration notifications" section. The previous all-or-nothing master toggle is replaced by three independent toggles.
- **No server-side or REST API changes required**; this is purely a client-side preferences store and UI update.

## Technical details

- Replaces the `showCollaborationNotifications` preference key with three independent keys: `showCollaborationJoinNotifications`, `showCollaborationLeaveNotifications`, and `showCollaborationPostSaveNotifications`.
- Introduces the `editor.CollaborationNotificationPreferenceDefaults` filter in `packages/edit-post/src/index.js` and `packages/edit-site/src/index.js` during `initializeEditor()`. The filter receives a defaults object and an editor context string (`core/edit-post` or `core/edit-site`).
- Updates `packages/editor/src/components/collaborators-presence/use-collaborator-notifications.ts` to read each preference independently from the `preferencesStore` and gate awareness hooks accordingly. Unset preferences default to `true`.
- Example usage:
```js
import { addFilter } from '@wordpress/hooks';

addFilter(
  'editor.CollaborationNotificationPreferenceDefaults',
  'my-plugin/collab-notif-defaults',
  ( defaults, editor ) => {
    if ( editor !== 'core/edit-post' ) return defaults;
    return {
      ...defaults,
      showCollaborationJoinNotifications: false,
      showCollaborationLeaveNotifications: true,
      showCollaborationPostSaveNotifications: true,
    };
  }
);
```

## Contribution

@shekharnwagh opened PR #79184 to address issue #76499. During review, @pkevan and @smithjw1 debated whether to collapse the toggles down to two (presence vs. updates) or retain three for maximum flexibility. The team agreed that maintaining three separate controls best accommodates edge-case workflows, leading to the final merged structure.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

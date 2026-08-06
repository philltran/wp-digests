# Post list tables row headers changed

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Joe Dolson
- **Published:** 2026-08-03
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/08/03/post-list-tables-row-headers-changed/](https://make.wordpress.org/core/2026/08/03/post-list-tables-row-headers-changed/)
- **Usefulness:** 4/5

## Summary

In WordPress 7.1, the primary row header (`<th scope="row">`) in post list tables moves from the checkbox column to the post title column. This structural shift ensures screen readers consistently announce the post name as the row identifier, rather than a checkbox that may be absent for non-editable posts. The change improves accessibility compliance and aligns list table semantics with standard HTML table practices.

## Impact

- **Plugin & theme developers (admin UI extensions):** Custom CSS or JavaScript targeting `.check-column` or expecting post titles/row actions inside `<td>` elements will break. Update selectors to target `th.check-column` or `th input[type="checkbox"]` for checkboxes, and `td.title`, `td.column-title`, `td.page-title`, `td.column-primary`, `td .row-title`, `td .post-state`, or `td .row-actions` for titles and actions.
- **Hosting & platform teams:** No action required for standard deployments, but custom admin themes or heavily modified list tables may need CSS/JS adjustments.
- **Backward compatibility:** Retaining both `td` and `th` variants in custom selectors will maintain compatibility with pre-7.1 versions.

## Technical details

The change restructures the DOM for post list table rows: the first column switches from `<th scope="row" class="check-column">` to `<td class="check-column">`, while the second column switches from `<td class="title column-title column-primary page-title">` to `<th scope="row" class="title column-title column-primary page-title" aria-label="...">`. The `aria-label` attribute is dynamically populated with the post title. Additionally, the CSS governing collapsed table cells in responsive viewports has been migrated to a flex layout.

Before:
```html
<tr>
  <th scope="row" class="check-column">
    <input type="checkbox" name="post[]" value="123">
  </th>
  <td class="title column-title column-primary page-title">
    <a class="row-title" href="...">Hello world!</a>
  </td>
</tr>
```

After:
```html
<tr>
  <td class="check-column">
    <input type="checkbox" name="post[]" value="123">
  </td>
  <th scope="row" class="title column-title column-primary page-title" aria-label="Hello world!">
    <a class="row-title" href="...">Hello world!</a>
  </th>
</tr>
```

## Contribution

Developed for the 7.1 cycle under ticket #32892 and reviewed by @wildworks, the change moved through review without recorded design debate or rejected alternatives in the source material.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*

# Test

This page confirms the documentation pipeline is working. If you can read this, the docs are building and rendering correctly.

## What a successful render confirms

When this page renders successfully, it tells you three things:

1. **The docs directory is wired up.** The build process found `docs/test.md` and included it in the output. If the file were missing or misnamed, you'd see a 404 instead of this page.

2. **Navigation is configured.** This page appears in the sidebar because it has an entry in `docs/nav.json`. Every documentation page needs a corresponding entry there to be discoverable.

3. **Markdown rendering works.** The heading above, this numbered list, and the inline code formatting all confirm that the Markdown pipeline is converting `.md` files into styled HTML as expected.

## Add a new documentation page

Each page in the Atom docs is a Markdown file inside the `docs/` directory. To add a new page:

1. Create a `.md` file in `docs/` (for example, `docs/my-page.md`).
2. Add an entry to the `nav` array in `docs/nav.json` with a `title` and `path`.

```json
{
  "nav": [
    { "title": "Introduction", "path": "introduction" },
    { "title": "My New Page", "path": "my-page" }
  ]
}
```

Each entry takes two fields:

- **`title`** (string) — The label shown in the sidebar navigation.
- **`path`** (string) — The filename inside `docs/` without the `.md` extension. For `docs/my-page.md`, use `"my-page"`.

The order of entries in the array controls the order pages appear in the sidebar. No restart or cache clear is needed after adding a page.

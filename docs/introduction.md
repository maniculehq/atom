# Atom — Headless CMS for Next.js

Atom is a headless CMS built specifically for Next.js. You write and manage your content — projects and posts in Markdown/MDX — through the Atom dashboard, then pull that content into your own Next.js site using the `atom-nextjs` npm package. The SDK provides async React server components that fetch posts from the Atom API, compile MDX, and render the result — no custom API calls or data-fetching boilerplate required.

## Understand projects and project keys

In Atom, a **project** is a named container for your blog posts. Each project gets a unique **project key** — a Bearer token that identifies which project's content to fetch. You copy that key from the dashboard and drop it into your Next.js app as an environment variable. From there, the `atom-nextjs` SDK handles everything: fetching posts from the Atom API, compiling your MDX, and rendering the result as React server components.

## Add a blog to your Next.js app

Before writing any code, create an account and set up your first project at the [Atom dashboard](https://cmsatom.netlify.app) to get your project key.

**Step 1 — Install the SDK:**

```bash
npm install atom-nextjs@latest
```

> **Tailwind CSS required.** The SDK's components use Tailwind utility classes for layout and typography. Your Next.js project must have [Tailwind CSS configured](https://tailwindcss.com/docs/guides/nextjs) or the rendered output will be unstyled.

**Step 2 — Add your project key to `.env.local`:**

```bash
ATOM_PROJECT_KEY=atom-your-project-key-here
```

**Step 3 — Render your blog pages.**

Because `AtomPage` and `Atom` are async server components, wrap them in `Suspense` to show a loading state while content arrives.

`AtomPage` fetches all posts in your project and renders a linked card grid. Use it for your blog index:

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function Blog() {
  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage
        projectKey={process.env.ATOM_PROJECT_KEY!}
        baseRoute="/blog"
      />
    </Suspense>
  );
}
```

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `projectKey` | `string` | Yes | — | Your project's Bearer token, copied from the dashboard |
| `baseRoute` | `string` | Yes | — | The URL prefix used to build each post's link, e.g. `"/blog"` |
| `title` | `boolean` | No | `true` | Whether to render the project name as an `<h1>` above the post grid |

`Atom` fetches and renders a single post — title, cover image, author, date, and the compiled MDX body. Use it for individual post pages:

```tsx
// app/blog/[id]/page.tsx
import { Atom, AtomArticleSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function BlogPost({ params }: { params: { id: string } }) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom
        projectKey={process.env.ATOM_PROJECT_KEY!}
        postId={params.id}
      />
    </Suspense>
  );
}
```

| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `projectKey` | `string` | Yes | — | Your project's Bearer token, copied from the dashboard |
| `postId` | `string` | Yes | — | The ID of the post to fetch, typically sourced from route params |
| `remarkPlugins` | `any[]` | No | `[]` | Additional [remark](https://github.com/remarkjs/remark) plugins passed to the MDX compiler. `remark-gfm` is always included. |
| `rehypePlugins` | `any[]` | No | `[]` | Additional [rehype](https://github.com/rehypejs/rehype) plugins passed to the MDX compiler. `rehype-sanitize` is always included. |

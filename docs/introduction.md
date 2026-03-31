# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

This page explains how Atom's two parts — the dashboard and the SDK — work together, and walks you through the two files you need to add a blog to your Next.js site.

## How the dashboard and SDK fit together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components (`AtomPage` to list posts and `AtomPost` to display a single post), pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

## Add a blog in two files

Install the SDK:

```bash
npm install atom-nextjs
```

Store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`), then create two route files.

**List all posts** — this page renders a linked card for every post in your project:

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function Blog() {
  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
    </Suspense>
  );
}
```

**Display a single post** — this page fetches and renders one post by its ID:

```tsx
// app/blog/[id]/page.tsx
import { AtomPost, AtomArticleSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function BlogPage({ params }: { params: { id: string } }) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <AtomPost projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

`AtomPage` and `AtomPost` are async server components that fetch from the Atom API. Wrapping them in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without it, rendering blocks until the fetch completes and there is no loading state.

### Component props

**`AtomPage`** — lists all posts in a project.

| Prop | Type | Description |
|------|------|-------------|
| `projectKey` | `string` | Your project's API key, used to authenticate requests to the Atom API. |
| `baseRoute` | `string` | The base path for post links (e.g. `"/blog"` generates links like `/blog/post-id`). |

**`AtomPost`** — renders a single post.

| Prop | Type | Description |
|------|------|-------------|
| `projectKey` | `string` | Your project's API key, used to authenticate requests to the Atom API. |
| `postId` | `string` | The unique identifier of the post to fetch and render. |

Each component also has a companion loading skeleton (`AtomLoadingSkeleton` and `AtomArticleSkeleton`) designed to be passed as a `<Suspense>` fallback.

## What Atom is (and isn't) responsible for

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else.

Your site stays in full control of layout, styling, and routing. Atom handles the content.

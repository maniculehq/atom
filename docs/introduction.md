# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app — with no lock-in on layout, styling, or routing.

## How Atom delivers content to your Next.js app

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* — a named collection of posts. You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components — `AtomPage` to list posts and `AtomPost` to display a single post — pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

```bash
npm install atom-nextjs
```

## Set up a blog with two files

Store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`) and reference it in each component.

**List all posts** — renders a linked card for every post in your project:

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

**Display a single post** — fetches and renders one post by its ID:

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

### Component props

`AtomPage` and `AtomPost` are async server components that fetch from the Atom API.

| Component | Prop | Type | Description |
|-----------|------|------|-------------|
| `AtomPage` | `projectKey` | `string` | Your project's API key (required) |
| `AtomPage` | `baseRoute` | `string` | URL prefix for post links, e.g. `"/blog"` (required) |
| `AtomPost` | `projectKey` | `string` | Your project's API key (required) |
| `AtomPost` | `postId` | `string` | The ID of the post to render (required) |

### Why wrap in Suspense?

Wrapping these components in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without `<Suspense>`, rendering blocks until the fetch completes and users see no loading state. The SDK exports `AtomLoadingSkeleton` and `AtomArticleSkeleton` as ready-made fallbacks, but you can pass any React node.

## What Atom owns — and what you own

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else — layout, styling, routing, and deployment.

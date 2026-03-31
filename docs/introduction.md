# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

By the end of this page you'll understand how Atom's two parts connect, and you'll have a working blog route in under ten minutes.

## How the dashboard and SDK work together

The system has two parts that communicate through a project key.

**The Atom dashboard** is where you create a *project* — a named collection of posts. You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key is a Bearer token: your app sends it in the `Authorization` header to authenticate every request to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. It provides two server components — `AtomPage` (lists posts) and `AtomPost` (displays a single post) — that fetch and render content directly from the Atom API at build or request time.

## Set up a blog route in two files

Install the SDK:

```bash
npm install atom-nextjs
```

Store your project key in an environment variable (`ATOM_PROJECT_KEY` in `.env.local`), then create two route files.

**List every post** — this page renders a linked card for each post in your project:

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function Blog() {
  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage
        baseRoute="/blog"                            // URL prefix for post links
        projectKey={process.env.ATOM_PROJECT_KEY!}   // Bearer token from the dashboard
      />
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
      <AtomPost
        projectKey={process.env.ATOM_PROJECT_KEY!}   // Bearer token from the dashboard
        postId={params.id}                            // dynamic segment from the URL
      />
    </Suspense>
  );
}
```

### Why Suspense?

`AtomPage` and `AtomPost` are async server components that fetch from the Atom API. Wrapping them in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without it, rendering blocks until the fetch completes and there is no loading state.

## Component reference

| Component | Props | Type | Description |
|-----------|-------|------|-------------|
| `AtomPage` | `projectKey` | `string` | Bearer token issued by the Atom dashboard. |
| | `baseRoute` | `string` | URL prefix used to build links to individual posts (e.g. `"/blog"`). |
| `AtomPost` | `projectKey` | `string` | Bearer token issued by the Atom dashboard. |
| | `postId` | `string` | ID of the post to fetch and render. |
| `AtomLoadingSkeleton` | — | — | Placeholder skeleton shown while `AtomPage` streams. |
| `AtomArticleSkeleton` | — | — | Placeholder skeleton shown while `AtomPost` streams. |

## What Atom owns — and what you own

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else — layout, styling, routing, and deployment.

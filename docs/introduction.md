# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the dashboard and SDK work together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components, `AtomPage` to list posts and `AtomPost` to display a single post, then pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

```bash
npm install atom-nextjs
```

## Add a blog to your Next.js app

A blog route is just two files. Store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`) and reference it in each component.

### List all posts with `AtomPage`

`AtomPage` fetches every published post in your project and renders a linked card for each one.

```tsx
// app/blog/page.tsx - list of all posts
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

| Prop | Type | Description | Default |
|------|------|-------------|---------|
| `projectKey` | `string` | The `project_key` from your Atom dashboard. Authenticates API requests. | *required* |
| `baseRoute` | `string` | Path prefix for post links (e.g. `"/blog"`, so each card links to `/blog/:id`). | *required* |

### Display a single post with `AtomPost`

`AtomPost` fetches and renders a single post by its ID, which comes from the dynamic route segment.

```tsx
// app/blog/[id]/page.tsx - single post view
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

| Prop | Type | Description | Default |
|------|------|-------------|---------|
| `projectKey` | `string` | The `project_key` from your Atom dashboard. Authenticates API requests. | *required* |
| `postId` | `string` | The post identifier, typically sourced from a dynamic route param like `params.id`. | *required* |

### Why `<Suspense>` matters here

`AtomPage` and `AtomPost` are async server components that fetch from the Atom API. Wrapping them in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without it, rendering blocks until the fetch completes and there is no loading state.

## What Atom is (and isn't) responsible for

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else.

| Atom manages | Your Next.js app manages |
|---|---|
| Post creation, editing, publishing | Routing, layouts, styling |
| Markdown content storage | Component rendering and SSR/SSG |
| Project keys and API authentication | Environment variables and deployment |
| Content delivery API | Everything else about your site |

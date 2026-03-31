# Add a blog to your Next.js app with Atom

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the dashboard and SDK work together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components — `AtomPage` to list posts and `AtomPost` to display a single post — then pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

## Get a blog running in two files

Install the SDK and store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`):

```bash
npm install atom-nextjs
```

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

**Display a single post** — this page fetches and renders one post by its route parameter:

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

`AtomPage` renders a list of post cards, each linking to an individual post route:

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `projectKey` | `string` | yes | The `project_key` from your Atom dashboard. Authenticates API requests. |
| `baseRoute` | `string` | yes | The route prefix for post links (e.g. `"/blog"` produces `/blog/{post-id}` links). |

`AtomPost` fetches and renders a single post by ID:

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `projectKey` | `string` | yes | The `project_key` from your Atom dashboard. Authenticates API requests. |
| `postId` | `string` | yes | The post identifier, typically sourced from a dynamic route param like `params.id`. |

### Why Suspense matters here

`AtomPage` and `AtomPost` are async server components that fetch from the Atom API. Wrapping them in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without it, rendering blocks until the fetch completes and there is no loading state.

### Generate SEO metadata for posts

The SDK also exports a `generatePostMetadata` helper that fetches a post's title and description so Next.js can set `<head>` metadata for each post page:

```tsx
// add to app/blog/[id]/page.tsx
import { generatePostMetadata } from 'atom-nextjs';

export const generateMetadata = async ({ params }: { params: { id: string } }) => {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
};
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `projectKey` | `string` | Your project's API key. |
| `postId` | `string` | The ID of the post to generate metadata for. |

This returns a Next.js-compatible `Metadata` object, so you can return it directly from `generateMetadata`.

## What Atom owns — and what you own

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else — layout, styling, routing, and deployment.

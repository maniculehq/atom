# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the two parts fit together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. It provides server components that fetch and render your content directly from the Atom API at build or request time:

| Export | Type | Purpose |
|---|---|---|
| `AtomPage` | Server component | Lists all posts in a project as linked cards |
| `AtomPost` | Server component | Fetches and renders a single post by ID |
| `AtomLoadingSkeleton` | Component | Loading placeholder for `AtomPage` |
| `AtomArticleSkeleton` | Component | Loading placeholder for `AtomPost` |
| `generatePostMetadata` | Async function | Returns Next.js `Metadata` for a post (title, description, open graph) |

## Get your blog running in two files

Install the SDK:

```bash
npm install atom-nextjs
```

Store your project key in an environment variable (`ATOM_PROJECT_KEY` in `.env.local`), then create two route files.

**List all posts** (`app/blog/page.tsx`):

```tsx
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

`AtomPage` props:

| Prop | Type | Description |
|---|---|---|
| `projectKey` | `string` | Your project's API key (from the Atom dashboard) |
| `baseRoute` | `string` | The route prefix used to link to individual posts (e.g. `"/blog"` links to `"/blog/[id]"`) |

**Display a single post** (`app/blog/[id]/page.tsx`):

```tsx
import { AtomPost, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { Suspense } from 'react';

// Generates <title>, Open Graph tags, etc. from the post content
export const generateMetadata = async ({ params }: { params: { id: string } }) => {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
};

export default function BlogPage({ params }: { params: { id: string } }) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <AtomPost projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

`AtomPost` props:

| Prop | Type | Description |
|---|---|---|
| `projectKey` | `string` | Your project's API key |
| `postId` | `string` | The ID of the post to fetch (typically from the route parameter) |

## Why the Suspense boundary matters

`AtomPage` and `AtomPost` are async server components that fetch from the Atom API. Wrapping them in `<Suspense>` lets Next.js stream the page shell immediately and show a skeleton while the fetch completes. Without it, the entire page blocks on the network request and the user sees nothing until data arrives.

## What Atom owns (and what it doesn't)

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else: layout, styling, routing, and deployment.

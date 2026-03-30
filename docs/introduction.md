# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the two parts fit together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components (`AtomPage` to list posts and `Atom` to display a single post), pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

## Get a blog running in two files

Install the SDK and store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`):

```bash
npm install atom-nextjs
```

Then create two route files — one to list every post, one to display a single post:

```tsx
// app/blog/page.tsx — linked card list of all posts in your project
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

```tsx
// app/blog/[id]/page.tsx — fetches and renders a single post by its ID
import { Atom, AtomArticleSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function BlogPage({ params }: { params: { id: string } }) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

`AtomPage` and `Atom` are async server components that fetch from the Atom API. Wrapping them in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without `<Suspense>`, rendering blocks until the fetch completes and there is no loading state.

### Component props

**`AtomPage`** — lists every post in a project.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | *required* | The project key from your Atom dashboard. |
| `baseRoute` | `string` | *required* | URL prefix for post links (e.g. `"/blog"`). Each card links to `{baseRoute}/{post.id}`. |
| `title` | `boolean` | `true` | Whether to render the project title as an `<h1>` above the post list. |

**`Atom`** — renders a single post.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | *required* | The project key from your Atom dashboard. |
| `postId` | `string` | *required* | The post ID, typically from a dynamic route param. |
| `remarkPlugins` | `any[]` | `[]` | Additional remark plugins passed to the markdown processor. |
| `rehypePlugins` | `any[]` | `[]` | Additional rehype plugins passed to the markdown processor. |

### Utility exports

Beyond the two main components, `atom-nextjs` exports helpers you can use directly.

**Metadata and SEO**

| Export | Purpose |
|--------|---------|
| `generatePostMetadata(projectKey, postId)` | Returns a Next.js `Metadata` object for a post — useful in `generateMetadata` for SEO. |
| `generateSitemap(projectKey)` | Returns sitemap entries for all posts in a project. |

**Data fetching**

| Export | Purpose |
|--------|---------|
| `getPost(projectKey, postId)` | Fetches a single post. Returns `{ response, success, message }`. |
| `getProject(projectKey)` | Fetches a project and all its posts. Returns `{ response, success, message }`. |

## What Atom handles — and what your app owns

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else — layout, styling, routing, and deployment.

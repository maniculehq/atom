# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the dashboard and SDK work together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value that authenticates your app's requests to the Atom API).

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components (`AtomPage` to list posts and `Atom` to display a single post), pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

## Get a blog running in two files

> **Prerequisites:** The SDK's built-in components use [Tailwind CSS](https://tailwindcss.com/) utility classes and the [`@tailwindcss/typography`](https://tailwindcss.com/docs/typography-plugin) plugin for article prose. Make sure both are configured in your Next.js project before continuing.

### Install the SDK

```bash
npm install atom-nextjs
```

Store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`).

### Create two route files

One file lists every post, the other displays a single post:

```tsx
// app/blog/page.tsx – linked card list of all posts in your project
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
// app/blog/[id]/page.tsx – fetches and renders a single post by its ID
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

## Component reference

### `AtomPage` – list every post in a project

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | *required* | The project key from your Atom dashboard. |
| `baseRoute` | `string` | *required* | URL prefix for post links (e.g. `"/blog"`). Each card links to `{baseRoute}/{post.id}`. |
| `title` | `boolean` | `true` | Whether to render the project title as an `<h1>` above the post list. |

### `Atom` – render a single post

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | *required* | The project key from your Atom dashboard. |
| `postId` | `string` | *required* | The post ID, typically from a dynamic route param. |
| `remarkPlugins` | `any[]` | `[]` | Additional remark plugins appended after the built-in `remarkGfm`. |
| `rehypePlugins` | `any[]` | `[]` | Additional rehype plugins appended after the built-in `rehypeSanitize`. |

The markdown processor always includes [`remark-gfm`](https://github.com/remarkjs/remark-gfm) (GitHub Flavored Markdown) and [`rehype-sanitize`](https://github.com/rehypejs/rehype-sanitize) (HTML sanitization). Any plugins you pass are added after these defaults.

### `AtomPostCard` – render a single post card

Used internally by `AtomPage`, but exported so you can build custom list layouts.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `post` | `ClientPost` | *required* | A post object from the project's `posts` array. |
| `baseRoute` | `string` | *required* | URL prefix for the post link. |

`ClientPost` has the following fields:

| Field | Type |
|-------|------|
| `id` | `string` |
| `title` | `string` |
| `author` | `string` |
| `teaser` | `string` |
| `image` | `string \| null` (optional) |
| `createdAt` | `Date` |
| `updatedAt` | `Date` |

### `AtomBody` – render markdown to HTML

Used internally by `Atom`, but exported so you can render arbitrary markdown.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `body` | `string` | *required* | Raw markdown/MDX string to render. |
| `className` | `string` | `undefined` | CSS class applied to the wrapper `<div>`. |
| `remarkPlugins` | `any[]` | `[]` | Additional remark plugins appended after `remarkGfm`. |
| `rehypePlugins` | `any[]` | `[]` | Additional rehype plugins appended after `rehypeSanitize`. |

## Utility exports

Beyond the components, `atom-nextjs` exports helpers you can use directly.

### Metadata and SEO

| Export | Signature | Purpose |
|--------|-----------|---------|
| `generatePostMetadata` | `(projectKey: string, postId: string) → Promise<Metadata>` | Returns a Next.js `Metadata` object for a post. Useful inside a route's `generateMetadata` function. |
| `generateSitemap` | `(projectKey: string, blogRoute: string) → Promise<Array<{ url, lastModified, priority }>>` | Returns sitemap entries for all posts in a project, plus an entry for the blog index route itself. Compatible with Next.js `MetadataRoute.Sitemap`. `blogRoute` is the full base URL (e.g. `"https://example.com/blog"`). |

### Data fetching

| Export | Signature | Purpose |
|--------|-----------|---------|
| `getPost` | `(projectKey: string, postId: string) → Promise<{ response, success, message }>` | Fetches a single post. `response` contains the post object when `success` is `true`. |
| `getProject` | `(projectKey: string) → Promise<{ response, success, message }>` | Fetches a project and all its posts. `response` contains the project with a `posts` array of `ClientPost` objects when `success` is `true`. |

## What Atom handles, and what your app owns

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else (layout, styling, routing, and deployment).

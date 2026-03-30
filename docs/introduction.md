# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the dashboard and SDK work together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token — an `Authorization` header value that authenticates your app's requests to the Atom API.

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

#### `ClientPost` shape

Every post returned by the Atom API conforms to this shape. You'll encounter it when using `AtomPostCard` directly or when calling [`getProject`](#getproject--fetch-a-project-and-all-its-posts) to build a custom list.

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique post identifier, used in URLs and API calls. |
| `title` | `string` | The post's display title. |
| `author` | `string` | Author name set in the dashboard. |
| `teaser` | `string` | Short summary shown on post cards. |
| `image` | `string \| null \| undefined` | Cover image URL. `null` or `undefined` when no image is set. |
| `createdAt` | `Date` | Timestamp when the post was first created. |
| `updatedAt` | `Date` | Timestamp of the most recent edit. |

### `AtomBody` – render markdown to HTML

Used internally by `Atom`, but exported so you can render arbitrary markdown. This is useful when you fetch post content with `getPost` and want full control over the surrounding layout.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `body` | `string` | *required* | Raw markdown/MDX string to render. |
| `className` | `string` | `undefined` | CSS class applied to the wrapper `<div>`. |
| `remarkPlugins` | `any[]` | `[]` | Additional remark plugins appended after `remarkGfm`. |
| `rehypePlugins` | `any[]` | `[]` | Additional rehype plugins appended after `rehypeSanitize`. |

## Utility exports

Beyond the components, `atom-nextjs` exports helpers you can use directly for SEO, sitemaps, and custom data fetching.

### `generatePostMetadata` – add SEO metadata to a post page

Returns a Next.js `Metadata` object for a single post. Call it from your route's `generateMetadata` function so each post page gets its own `title`, `description`, `keywords`, and `authors` fields.

**Signature:** `(projectKey: string, postId: string) → Promise<Metadata>`

```tsx
// app/blog/[id]/page.tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { Suspense } from 'react';

export async function generateMetadata({ params }: { params: { id: string } }) {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
}

export default function BlogPage({ params }: { params: { id: string } }) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

### `generateSitemap` – create sitemap entries for all posts

Returns sitemap entries for every post in a project, plus an entry for the blog index route. The output is compatible with Next.js `MetadataRoute.Sitemap`.

**Signature:** `(projectKey: string, blogRoute: string) → Promise<Array<{ url: string, lastModified: Date, priority: number }>>`

`blogRoute` should be the full base URL, e.g. `"https://example.com/blog"`.

```tsx
// app/sitemap.ts
import { generateSitemap } from 'atom-nextjs';

export default async function sitemap() {
  return generateSitemap(process.env.ATOM_PROJECT_KEY!, 'https://example.com/blog');
}
// Returns:
// [
//   { url: "https://example.com/blog/post-id-1", lastModified: Date, priority: 0.5 },
//   ...
//   { url: "https://example.com/blog", lastModified: Date, priority: 0.6 },
// ]
```

### `getPost` – fetch a single post

Fetches a single post by ID. Use this when you need the raw post data — for example, to build a fully custom detail page instead of using the `Atom` component.

**Signature:** `(projectKey: string, postId: string) → Promise<ApiResponse<Post>>`

The return type is `ApiResponse<Post>`: `{ response: Post, success: boolean, message: string }`. When `success` is `true`, `response` contains the post object. When `success` is `false`, `response` is `null` and `message` describes the error.

### `getProject` – fetch a project and all its posts

Fetches a project and all its posts. Use this when you want to build a completely custom list page instead of relying on `AtomPage`.

**Signature:** `(projectKey: string) → Promise<ApiResponse<ClientProject>>`

The return type is `ApiResponse<ClientProject>`: `{ response: ClientProject, success: boolean, message: string }`. When `success` is `true`, `response` contains the project with a `posts` array of [`ClientPost`](#clientpost-shape) objects. When `success` is `false`, `response` is `null` and `message` describes the error.

## What Atom handles, and what your app owns

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else — layout, styling, routing, and deployment.

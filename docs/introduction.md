# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the two parts fit together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components (`AtomPage` to list posts and `Atom` to display a single post), pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

## Get a blog running in two files

### Prerequisites

The SDK's built-in components use [Tailwind CSS](https://tailwindcss.com/) utility classes for layout and styling, plus the [`@tailwindcss/typography`](https://tailwindcss.com/docs/typography-plugin) plugin for article prose formatting. Make sure both are configured in your Next.js project before continuing.

### Installation

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
| `remarkPlugins` | `any[]` | none | Additional remark plugins appended after the built-in `remarkGfm`. |
| `rehypePlugins` | `any[]` | none | Additional rehype plugins appended after the built-in `rehypeSanitize`. |

The markdown processor always includes [`remark-gfm`](https://github.com/remarkjs/remark-gfm) (GitHub Flavored Markdown) and [`rehype-sanitize`](https://github.com/rehypejs/rehype-sanitize) (HTML sanitization). Any plugins you pass are added after these defaults.

**`AtomPostCard`** — renders a single post card (used internally by `AtomPage`, but exported for custom layouts).

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `post` | `ClientPost` | *required* | A post object from the project's `posts` array. |
| `baseRoute` | `string` | *required* | URL prefix for the post link. |

**`AtomBody`** — renders a markdown string to HTML (used internally by `Atom`, but exported for custom layouts).

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `body` | `string` | *required* | Raw markdown/MDX string to render. |
| `className` | `string` | none | CSS class applied to the wrapper `<div>`. |
| `remarkPlugins` | `any[]` | none | Additional remark plugins appended after `remarkGfm`. |
| `rehypePlugins` | `any[]` | none | Additional rehype plugins appended after `rehypeSanitize`. |

### Utility exports

Beyond the components, `atom-nextjs` exports helpers you can use directly.

**Metadata and SEO**

| Export | Purpose |
|--------|---------|
| `generatePostMetadata(projectKey, postId)` | Returns a Next.js `Metadata` object for a post — useful in `generateMetadata` for SEO. |
| `generateSitemap(projectKey, blogRoute)` | Returns sitemap entries for all posts in a project. `blogRoute` is the base URL (e.g. `"https://example.com/blog"`). |

**Data fetching**

| Export | Purpose |
|--------|---------|
| `getPost(projectKey, postId)` | Fetches a single post. Returns `{ response, success, message }`. |
| `getProject(projectKey)` | Fetches a project and all its posts. Returns `{ response, success, message }`. |

## What Atom handles — and what your app owns

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else — layout, styling, routing, and deployment.

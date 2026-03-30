# Next.js SDK Guide

Atom is a headless CMS built for Next.js. You write and manage blog posts in the Atom dashboard, then pull that content into your own Next.js site using the `atom-nextjs` SDK. The SDK gives you ready-made React server components that fetch your posts and render them as styled pages, with no client-side JavaScript shipped to your visitors.

This guide walks you through everything: installing the SDK, creating your first blog listing, rendering individual posts, adding SEO metadata, and handling loading states. By the end, you'll have a fully working blog powered by Atom.

**Quick install:**

```bash
npm i atom-nextjs@latest @tailwindcss/typography
```

```tsx
// app/blog/page.tsx — minimal blog listing
import { AtomPage } from 'atom-nextjs';

export default function Blog() {
  return (
    <AtomPage
      baseRoute="/blog"
      projectKey={process.env.ATOM_PROJECT_KEY!}
    />
  );
}
```

---

## Prerequisites

Before you start, make sure you have:

- A **Next.js 14+ project** using the App Router (the SDK relies on React server components)
- **Tailwind CSS** set up in your project
- An **Atom account** with at least one project created at [cmsatom.netlify.app](https://cmsatom.netlify.app). You'll need the project key from your dashboard.

Your project key is a Bearer token that looks like `atom-aBcDeFgHiJkLmN...`. You can find it on your project's page in the Atom dashboard. Keep it secret, because anyone with this key can read your content through the API.

## Step 1: Install the SDK packages

The SDK needs two packages: `atom-nextjs` itself and the Tailwind CSS typography plugin (which styles the rendered markdown).

```bash
npm i atom-nextjs@latest @tailwindcss/typography
```

## Step 2: Configure Tailwind to include SDK styles

The SDK ships its own components with Tailwind classes. For those classes to work, Tailwind needs to scan the SDK's source files. You also need to add the typography plugin so rendered markdown gets proper styling (headings, paragraphs, code blocks, lists).

Update your `tailwind.config.ts` (or `tailwind.config.js`):

```ts
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    // Your existing content paths
    './app/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    // Add the SDK's components so Tailwind generates their classes
    './node_modules/atom-nextjs/src/components/*.{ts,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [
    // Add the typography plugin for markdown styling
    require('@tailwindcss/typography'),
  ],
};
```

The `@tailwindcss/typography` plugin provides the `prose` classes that the `Atom` component uses to style your markdown content. Without it, your blog posts will look unstyled.

## Step 3: Store your project key securely

Add your Atom project key to your environment variables. Create or update your `.env.local` file:

```bash
ATOM_PROJECT_KEY=atom-your-project-key-here
```

The SDK components run as server components, so this key never reaches the browser. It's only used in server-side `fetch` calls to the Atom API.

## Step 4: Build the blog listing page

The `AtomPage` component fetches all posts from your project and renders them as a grid of clickable cards. Each card shows the post's title, teaser, author, date, and cover image (if one exists).

Create `app/blog/page.tsx`:

```tsx
import { AtomPage } from 'atom-nextjs';
import { cookies } from 'next/headers';
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Blog',
};

export default function Blog() {
  // Call cookies() to opt out of Next.js static caching.
  // This ensures new posts appear immediately after publishing.
  const _cookies = cookies();

  return (
    <AtomPage
      baseRoute="/blog"
      projectKey={process.env.ATOM_PROJECT_KEY!}
    />
  );
}
```

### `AtomPage` props

| Prop | Type | Default | Description |
|---|---|---|---|
| `projectKey` | `string` | **(required)** | Your Atom project key, sent as a Bearer token to the Atom API. |
| `baseRoute` | `string` | **(required)** | URL path prefix for post links — `"/blog"` means post `abc123` links to `/blog/abc123`. |
| `title` | `boolean` | `true` | When `true`, renders your project title as an `<h1>` above the post grid; set to `false` to provide your own heading. |

### Why call `cookies()`?

Next.js aggressively caches server component output. Calling `cookies()` signals to Next.js that this page depends on request-time data, which disables static caching. Without it, your blog listing might show stale content after you publish or update a post.

If you prefer caching for performance and don't mind content updates being delayed, you can remove the `cookies()` call.

## Step 5: Build the single post page

The `Atom` component fetches a single post by its ID and renders the full article: title, cover image, author, publish date, and the markdown body.

Create `app/blog/[id]/page.tsx`:

```tsx
import { Atom, generatePostMetadata } from 'atom-nextjs';
import { cookies } from 'next/headers';

export type BlogParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogParams) => {
  const metadata = await generatePostMetadata(
    process.env.ATOM_PROJECT_KEY!,
    params.id
  );

  return metadata;
};

export default function BlogPage({ params }: BlogParams) {
  const _cookies = cookies();

  return (
    <Atom
      projectKey={process.env.ATOM_PROJECT_KEY!}
      postId={params.id}
    />
  );
}
```

There are two things happening here:

**The `Atom` component** takes `projectKey` and `postId`, fetches the post from the API, and renders a complete `<article>` with the `prose` Tailwind class for markdown styling.

### `Atom` props

| Prop | Type | Default | Description |
|---|---|---|---|
| `projectKey` | `string` | **(required)** | Your Atom project key. |
| `postId` | `string` | **(required)** | The ID of the post to fetch and render. |
| `remarkPlugins` | `Plugin[]` | `[]` | Additional [remark](https://github.com/remarkjs/remark) plugins, on top of the included `remark-gfm` for GitHub Flavored Markdown (tables, strikethrough, task lists). |
| `rehypePlugins` | `Plugin[]` | `[]` | Additional [rehype](https://github.com/rehypejs/rehype) plugins, on top of the included `rehype-sanitize` for XSS prevention. |

### Generate SEO metadata with `generatePostMetadata`

**The `generatePostMetadata` function** fetches the post data and returns a Next.js `Metadata` object with the post's title, description (from the teaser), keywords, and author. Next.js uses this to set `<title>`, `<meta>` description, and other SEO tags automatically.

The returned metadata has this shape:

```ts
// Return type: Metadata (from 'next')
{
  title: string;        // Post title
  description: string;  // Post teaser
  keywords?: string[];  // Post keywords
  authors: { name: string };  // Post author
}
```

## Step 6: Add loading skeletons

Since `Atom` and `AtomPage` are async server components that fetch data from the Atom API, there's a short delay before content appears. React's `Suspense` lets you show a loading skeleton during that fetch. The SDK provides two skeleton components that match the layout of the content they replace.

Update your blog listing page (`app/blog/page.tsx`):

```tsx
import { AtomLoadingSkeleton, AtomPage } from 'atom-nextjs';
import { cookies } from 'next/headers';
import { Metadata } from 'next';
import { Suspense } from 'react';

export const metadata: Metadata = {
  title: 'Blog',
};

export default function Blog() {
  const _cookies = cookies();

  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage
        baseRoute="/blog"
        projectKey={process.env.ATOM_PROJECT_KEY!}
      />
    </Suspense>
  );
}
```

And your single post page (`app/blog/[id]/page.tsx`):

```tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { cookies } from 'next/headers';
import { Suspense } from 'react';

export type BlogParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogParams) => {
  const metadata = await generatePostMetadata(
    process.env.ATOM_PROJECT_KEY!,
    params.id
  );

  return metadata;
};

export default function BlogPage({ params }: BlogParams) {
  const _cookies = cookies();

  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom
        projectKey={process.env.ATOM_PROJECT_KEY!}
        postId={params.id}
      />
    </Suspense>
  );
}
```

`AtomLoadingSkeleton` renders four placeholder cards that mimic the post grid layout. `AtomArticleSkeleton` renders a placeholder that mimics a full article page. Both use the `react-loading-skeleton` library, which is bundled with the SDK.

## Step 7: Generate a sitemap for search engines

Sitemaps help search engines discover and index your blog posts. The SDK's `generateSitemap` function fetches your project's posts and returns an array of sitemap entries in the format Next.js expects.

Create `app/sitemap.ts`:

```ts
import { MetadataRoute } from 'next';
import { generateSitemap } from 'atom-nextjs';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const routes = await generateSitemap(
    process.env.ATOM_PROJECT_KEY!,
    'https://yourdomain.com/blog'
  );

  return [
    {
      url: 'https://yourdomain.com',
      lastModified: new Date(),
      priority: 0.7,
    },
    ...routes,
  ];
}
```

Replace `https://yourdomain.com` with your actual domain. The function generates one entry for each post (at priority 0.5) and one for the blog index (at priority 0.6), using each post's `updatedAt` timestamp as the `lastModified` date.

Each entry in the returned array has this shape:

```ts
// Each item in the routes array
{
  url: string;          // e.g. "https://yourdomain.com/blog/abc123"
  lastModified: Date;   // Post's updatedAt timestamp
  priority: number;     // 0.5 for posts, 0.6 for the blog index
}
```

## Fetch data directly for custom layouts

If you need more control over how content is displayed — for example, building a custom post card or filtering posts — you can use the lower-level `getProject` and `getPost` functions instead of the pre-built components.

### Fetch all posts with `getProject`

```tsx
import { getProject } from 'atom-nextjs';

export default async function CustomBlog() {
  const res = await getProject(process.env.ATOM_PROJECT_KEY!);

  if (!res.success) {
    return <p>{res.message}</p>;
  }

  const project = res.response;

  return (
    <div>
      <h1>{project.title}</h1>
      {project.posts.map((post) => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.teaser}</p>
          <span>{post.author}</span>
        </div>
      ))}
    </div>
  );
}
```

`getProject` returns an `ApiResponse<ClientProject>` with this shape:

```ts
{
  success: boolean;
  message: string;
  response: {
    title: string;
    id: string;
    createdAt: Date;
    updatedAt: Date;
    posts: Array<{
      id: string;
      title: string;
      teaser: string;
      author: string;
      image?: string | null;
      createdAt: Date;
      updatedAt: Date;
    }>;
  };
}
```

Note that `ClientPost` objects don't include the full markdown `body`. This keeps the listing response lightweight. The body is only returned when you fetch a single post.

### Fetch a single post with `getPost`

```tsx
import { getPost } from 'atom-nextjs';

export default async function CustomPost({ postId }: { postId: string }) {
  const res = await getPost(process.env.ATOM_PROJECT_KEY!, postId);

  if (!res.success) {
    return <p>{res.message}</p>;
  }

  const post = res.response;

  return (
    <article>
      <h1>{post.title}</h1>
      <p>By {post.author}</p>
      {/* post.body is raw markdown — you'll need to render it yourself */}
      <div>{post.body}</div>
    </article>
  );
}
```

`getPost` returns an `ApiResponse<Post>` with this shape:

```ts
{
  success: boolean;
  message: string;
  response: {
    id: string;
    title: string;
    teaser: string;
    author: string;
    image: string | null;
    body: string;           // Full markdown content
    keywords?: string[];    // SEO keywords
    creator_uid: string;    // UID of the post creator
    createdAt: Date;
    updatedAt: Date;
  };
}
```

### Render markdown with `AtomBody`

If you fetch posts with `getPost` but still want the SDK's markdown rendering, use the `AtomBody` component:

```tsx
import { getPost, AtomBody } from 'atom-nextjs';

export default async function CustomPost({ postId }: { postId: string }) {
  const res = await getPost(process.env.ATOM_PROJECT_KEY!, postId);

  if (!res.success) {
    return <p>{res.message}</p>;
  }

  return (
    <article className="prose lg:prose-xl">
      <h1>{res.response.title}</h1>
      <AtomBody body={res.response.body} />
    </article>
  );
}
```

`AtomBody` is an async server component that compiles markdown to React elements using `next-mdx-remote`. It includes `remark-gfm` (GitHub Flavored Markdown) and `rehype-sanitize` (XSS protection) by default.

| Prop | Type | Default | Description |
|---|---|---|---|
| `body` | `string` | **(required)** | The raw markdown string to render. |
| `className` | `string` | `undefined` | CSS class name applied to the wrapper `<div>` around the rendered markdown. |
| `remarkPlugins` | `Plugin[]` | `[]` | Additional remark plugins (on top of `remark-gfm`). |
| `rehypePlugins` | `Plugin[]` | `[]` | Additional rehype plugins (on top of `rehype-sanitize`). |

## Handle dark mode

The SDK's built-in components currently do not support dark mode. The post cards use hardcoded light-mode colors (like `hover:bg-slate-50` and `text-slate-500`). If your site uses dark mode, you'll want to fetch data with `getProject`/`getPost` and build your own components as described in [Fetch data directly for custom layouts](#fetch-data-directly-for-custom-layouts).

## Understand rate limits

The Atom API limits requests to **30 per minute per IP address**. In normal usage this is unlikely to be an issue, since the SDK only makes one fetch per page render. But if you're generating many pages at build time (for example, with `generateStaticParams`), you might hit this limit. If that happens, add a small delay between builds or contact the Atom team.

## SDK exports at a glance

Here's a summary of everything the SDK exports:

| Export | Type | Purpose |
|---|---|---|
| `AtomPage` | Server component | Renders a grid of post cards for a project |
| `Atom` | Server component | Renders a full blog post article |
| `AtomBody` | Server component | Renders a markdown string to React elements |
| `AtomPostCard` | Component | A single post card (used internally by `AtomPage`) |
| `AtomLoadingSkeleton` | Component | Loading placeholder for post listings |
| `AtomArticleSkeleton` | Component | Loading placeholder for single post pages |
| `getProject` | Async function | Fetches project data (title and post summaries) |
| `getPost` | Async function | Fetches a single post with full markdown body |
| `generatePostMetadata` | Async function | Returns a Next.js `Metadata` object for a post |
| `generateSitemap` | Async function | Returns sitemap entries for all posts in a project |

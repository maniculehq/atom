# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app — with no lock-in on layout, styling, or routing. Two files and one API key is all it takes to go from an empty project to a working blog.

## How the two parts fit together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. It provides async server components that fetch and render your content directly from the Atom API at build or request time. Because these are server components, your `project_key` never leaks to the browser.

### SDK exports at a glance

**Components**

| Export | Type | Purpose |
|---|---|---|
| `AtomPage` | Async server component | Lists all posts in a project as linked cards |
| `AtomPost` | Async server component | Fetches and renders a single post by ID (MDX with `remark-gfm`) |
| `AtomPostCard` | Component | Individual post card used internally by `AtomPage` — also available for custom layouts |
| `AtomBody` | Component | Renders a raw MDX body string; useful if you fetch post data yourself |
| `AtomLoadingSkeleton` | Component | Loading placeholder for `AtomPage` |
| `AtomArticleSkeleton` | Component | Loading placeholder for `AtomPost` |

**Functions**

| Export | Purpose |
|---|---|
| `generatePostMetadata(projectKey, postId)` | Returns a Next.js `Metadata` object (title, description, keywords, author) for a post |
| `generateSitemap(projectKey, blogRoute)` | Returns an array of sitemap entries for all posts in a project |
| `getPost(projectKey, postId)` | Fetches a single post — returns `ApiResponse<Post>` |
| `getProject(projectKey)` | Fetches the full project with posts — returns `ApiResponse<ClientProject>` |

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

| Prop | Type | Default | Description |
|---|---|---|---|
| `projectKey` | `string` | *(required)* | Your project's API key (from the Atom dashboard) |
| `baseRoute` | `string` | *(required)* | Route prefix used to link to individual posts (e.g. `"/blog"` links to `"/blog/[id]"`) |
| `title` | `boolean` | `true` | Whether to render the project title as an `<h1>` above the post list |

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

| Prop | Type | Default | Description |
|---|---|---|---|
| `projectKey` | `string` | *(required)* | Your project's API key |
| `postId` | `string` | *(required)* | The ID of the post to fetch (typically from the route parameter) |
| `remarkPlugins` | `any[]` | `undefined` | Additional remark plugins to pass to the MDX compiler |
| `rehypePlugins` | `any[]` | `undefined` | Additional rehype plugins to pass to the MDX compiler |

## Add a sitemap for SEO

The SDK can generate sitemap entries for all posts in your project. Export a `default` function from `app/sitemap.ts` and Atom produces the routes:

```ts
import { generateSitemap } from 'atom-nextjs';

export default async function sitemap() {
  const blogRoutes = await generateSitemap(
    process.env.ATOM_PROJECT_KEY!,
    'https://yoursite.com/blog'
  );
  // blogRoutes is an array of { url, lastModified, priority }
  return [...blogRoutes];
}
```

## Why the Suspense boundary matters

`AtomPage` and `AtomPost` are async server components that `fetch()` from the Atom API during rendering. Wrapping them in `<Suspense>` lets Next.js stream the page shell immediately and show a skeleton while the fetch completes. Without it, the entire page blocks on the network request and the user sees nothing until data arrives.

## Response shapes

If you use the lower-level `getPost` or `getProject` functions instead of the pre-built components, every call returns an `ApiResponse<T>` wrapper:

```ts
type ApiResponse<T> = {
  response: T;
  success: boolean;
  message: string;
};
```

When using these lower-level functions, you can also inspect rate limiting headers from the HTTP response. The Atom API includes the following headers that help you monitor your usage:

- `X-RateLimit-Limit`: The maximum number of requests allowed per time window
- `X-RateLimit-Remaining`: The number of requests remaining in the current window
- `X-RateLimit-Reset`: Unix timestamp when the rate limit window resets
- `Retry-After`: Number of seconds to wait before retrying (only present when rate limited)

A `Post` (returned by `getPost`) has this shape:

```ts
type Post = {
  id: string;
  title: string;
  author: string;
  body: string;        // raw MDX/markdown string
  teaser: string;      // short summary
  image: string | null;
  keywords?: string[];
  creator_uid: string;
  createdAt: Date;
  updatedAt: Date;
};
```

A `ClientProject` (returned by `getProject`) contains the project metadata and an array of posts:

```ts
type ClientProject = {
  id: string;
  title: string;
  posts: ClientPost[]; // each post has id, title, teaser, author, image, createdAt, updatedAt
  createdAt: Date;
  updatedAt: Date;
};
```

## What Atom owns (and what it doesn't)

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else — layout, styling, routing, and deployment.
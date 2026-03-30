# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on routing or page layout.

## How the two parts fit together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components (`AtomPage` to list posts and `Atom` to display a single post), pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

## Add a blog to your Next.js app

Install the SDK:

```bash
npm install atom-nextjs
```

The SDK components use [Tailwind CSS](https://tailwindcss.com) utility classes, including `prose` from [`@tailwindcss/typography`](https://tailwindcss.com/docs/typography-plugin). Your Tailwind config needs both the typography plugin and the SDK's component paths in `content` so the classes are included in your build:

```js
// tailwind.config.js
module.exports = {
  content: [
    // ... your existing paths
    './node_modules/atom-nextjs/src/components/*.{ts,tsx}',
  ],
  plugins: [
    require('@tailwindcss/typography'),
  ],
};
```

Store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`), then create two route files.

**List all posts** — `AtomPage` fetches every post in your project and renders a linked card for each one:

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

`AtomPage` accepts these props:

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | *required* | Your project's API key |
| `baseRoute` | `string` | *required* | URL prefix for post links (e.g. `"/blog"` generates `"/blog/post-id"`) |
| `title` | `boolean` | `true` | Whether to render the project title as an `<h1>` above the post list |

**Display a single post** — `Atom` fetches one post by ID and renders the full article (title, cover image, author, date, and markdown body):

```tsx
// app/blog/[id]/page.tsx
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

`Atom` accepts these props:

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | *required* | Your project's API key |
| `postId` | `string` | *required* | The ID of the post to render |
| `remarkPlugins` | `any[]` | `[]` | Additional [remark](https://github.com/remarkjs/remark) plugins for markdown processing |
| `rehypePlugins` | `any[]` | `[]` | Additional [rehype](https://github.com/rehypejs/rehype) plugins for HTML processing |

### Why Suspense is used in both examples

`AtomPage` and `Atom` are async server components that fetch from the Atom API. Wrapping them in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without it, rendering blocks until the fetch completes and there is no loading state.

## Other exports you can use

Beyond the two main components, the `atom-nextjs` package exports several utilities for more advanced use cases:

| Export | Purpose |
|--------|---------|
| `getPost(projectKey, postId)` | Fetch a single post directly — useful when you need raw post data outside a component |
| `getProject(projectKey)` | Fetch the full project with all its posts — useful for custom list layouts |
| `generatePostMetadata(projectKey, postId)` | Generate Next.js `Metadata` for a post (title, description, keywords, authors) |
| `generateSitemap(projectKey, blogRoute)` | Generate sitemap entries for all posts in a project |
| `AtomBody` | Render just the markdown body of a post, with optional remark/rehype plugins |
| `AtomPostCard` | Render a single post card — the same card `AtomPage` uses internally |

## What Atom is (and isn't) responsible for

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else.

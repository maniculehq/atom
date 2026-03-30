# Introduction

Atom is a headless CMS built for Next.js. You write and manage blog posts in the Atom dashboard, then render them in your own Next.js app with a small companion package called `atom-nextjs`. Your app keeps full control over layout, styling, routing, and deployment. Atom handles the content.

## How the two parts fit together

The system has two pieces that communicate through a project key.

**The Atom dashboard** is where you create a _project_ (a named collection of posts). Each project gets a unique `project_key`, a Bearer token that authenticates requests from your app to the Atom API. Inside a project, you write posts in a markdown editor with support for titles, authors, cover images, keywords, and teaser text.

**The `atom-nextjs` SDK** is what you install in your Next.js site. It provides server components that fetch content from the Atom API at request time and render it using `next-mdx-remote`. The two main components are `AtomPage` (lists all posts in a project) and `Atom` (renders a single post). Because these are async server components, your project key never reaches the browser.

Here's how the data flows:

1. You sign into the dashboard, create a project, and write posts in markdown.
2. The dashboard stores your content in MongoDB and gives you a `project_key`.
3. In your Next.js app, you pass that key to the SDK's server components.
4. At render time, the SDK calls the Atom API with the key as a `Bearer` token, fetches the content, and compiles the markdown to HTML.

## Get a blog running in two files

### Prerequisites

- A Next.js 13+ project using the App Router
- [Tailwind CSS](https://tailwindcss.com/) configured in your project
- The [`@tailwindcss/typography`](https://tailwindcss.com/docs/typography-plugin) plugin installed (the SDK uses `prose` classes for article formatting)
- An Atom account with at least one project created (sign up at the dashboard to get your `project_key`)

### Installation

```bash
npm i atom-nextjs@latest @tailwindcss/typography
```

Store your project key in `.env.local`:

```
ATOM_PROJECT_KEY=atom-your-key-here
```

Then update your Tailwind config to include the SDK's component styles and the typography plugin:

```ts
// tailwind.config.ts
module.exports = {
  content: [
    // ... your existing paths
    './node_modules/atom-nextjs/src/components/*.{ts,tsx}',
  ],
  plugins: [
    // ... your existing plugins
    require('@tailwindcss/typography'),
  ],
};
```

### Create the blog list page

This page fetches all posts in your project and renders them as linked cards.

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export const metadata = { title: 'Blog' };

export default function Blog() {
  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
    </Suspense>
  );
}
```

`AtomPage` is an async server component. It calls the Atom API with your project key, gets the list of posts, and renders an `AtomPostCard` for each one. The `baseRoute` prop tells the cards where to link: each card points to `{baseRoute}/{post.id}`. Wrapping it in `<Suspense>` lets Next.js stream the page shell immediately while the fetch completes in the background.

### Create the single post page

This page fetches one post by its ID and renders the full article with compiled markdown.

```tsx
// app/blog/[id]/page.tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { Suspense } from 'react';

type BlogParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogParams) => {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
};

export default async function BlogPage({ params }: BlogParams) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

`Atom` fetches the post, then passes the markdown body through `next-mdx-remote` with `remark-gfm` (GitHub Flavored Markdown) and `rehype-sanitize` (HTML sanitization) applied by default. The `generatePostMetadata` helper populates the page's `<title>`, description, keywords, and author from the post data, so you get proper SEO without writing metadata logic yourself.

### Verify it works

Start your dev server (`npm run dev`), navigate to `/blog`, and you should see cards for each post in your Atom project. Clicking a card takes you to the full article at `/blog/{post-id}`.

## Component reference

### AtomPage

Lists every post in a project as a grid of linked cards.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | required | Your project's Bearer token from the Atom dashboard. |
| `baseRoute` | `string` | required | URL prefix for post links. Each card links to `{baseRoute}/{post.id}`. |
| `title` | `boolean` | `true` | When `true`, renders the project title as an `<h1>` above the cards. |

### Atom

Renders a single post as a full article with compiled markdown.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | required | Your project's Bearer token. |
| `postId` | `string` | required | The post ID, typically from a dynamic route param. |
| `remarkPlugins` | `any[]` | `[]` | Additional remark plugins, appended after the built-in `remarkGfm`. |
| `rehypePlugins` | `any[]` | `[]` | Additional rehype plugins, appended after the built-in `rehypeSanitize`. |

### AtomBody

Renders a raw markdown string to HTML. This is what `Atom` uses internally, but you can import it directly if you need to build a custom post layout.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `body` | `string` | required | The markdown string to render. |
| `className` | `string` | `undefined` | CSS class applied to the wrapper `<div>`. |
| `remarkPlugins` | `any[]` | `[]` | Additional remark plugins, appended after `remarkGfm`. |
| `rehypePlugins` | `any[]` | `[]` | Additional rehype plugins, appended after `rehypeSanitize`. |

### AtomPostCard

Renders a single post as a linked card. Used internally by `AtomPage`, but exported for custom layouts where you want to fetch the project yourself and render cards manually.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `post` | `ClientPost` | required | A post object from the project's `posts` array. |
| `baseRoute` | `string` | required | URL prefix for the post link. |

## Utility functions

### generatePostMetadata(projectKey, postId)

Fetches a post and returns a Next.js `Metadata` object containing the post's title, description (from the teaser), keywords, and author. Use this in your page's `generateMetadata` export for automatic SEO.

### generateSitemap(projectKey, blogRoute)

Fetches all posts in a project and returns an array of sitemap entries. Each entry includes the post URL (constructed from `blogRoute`), `lastModified` timestamp, and priority. Pass the result directly to your `sitemap.ts` route.

```ts
// app/sitemap.ts
import { generateSitemap } from 'atom-nextjs';

export default async function sitemap() {
  const blogEntries = await generateSitemap(
    process.env.ATOM_PROJECT_KEY!,
    'https://yoursite.com/blog'
  );

  return [
    { url: 'https://yoursite.com', lastModified: new Date() },
    ...blogEntries,
  ];
}
```

### getProject(projectKey)

Fetches a project and all its posts. Returns `{ response: ClientProject, success: boolean, message: string }`. The `ClientProject` type includes the project title and an array of `ClientPost` objects (without the full markdown body, just metadata like title, author, teaser, image, and timestamps).

### getPost(projectKey, postId)

Fetches a single post with its full markdown body. Returns `{ response: Post, success: boolean, message: string }`.

## What Atom handles, and what your app owns

Atom is focused on one thing: managing and delivering blog content through an API. It does not host your site, inject scripts into your pages, or constrain your app's UI. The boundary is clear:

**Atom owns:** content storage, the markdown editor, the REST API, project key authentication, and the `atom-nextjs` rendering components.

**Your app owns:** routing, layout, styling, deployment, caching strategy, and any custom UI around the blog content.

The SDK components are designed to drop into any layout. You can wrap `AtomPage` or `Atom` inside your own containers, apply your own CSS, or skip the pre-built components entirely and use the `getProject` / `getPost` functions to fetch raw data and render it however you want.

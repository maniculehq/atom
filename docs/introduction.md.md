# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the two parts fit together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components (`AtomPage` to list posts and `Atom` to display a single post), pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

```bash
npm install atom-nextjs @tailwindcss/typography
```

The `@tailwindcss/typography` plugin is required because the SDK's `Atom` component renders post bodies inside a `prose` class container, relying on Tailwind's typography styles for headings, paragraphs, lists, and code blocks.

## Setting up a blog in two files

Store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`) and reference it in each component.

First, the blog listing page:

```tsx
// app/blog/page.tsx — renders a linked card list of all posts in your project
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

`AtomPage` fetches all posts in your project via the Atom API and renders each one as a clickable card. The `baseRoute` prop tells the component where to link each card, so clicking a post navigates to `/blog/{post-id}`.

Next, the individual post page:

```tsx
// app/blog/[id]/page.tsx — fetches and renders a single post by its ID
import { Atom, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
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
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

`Atom` fetches a single post and renders its markdown body using `next-mdx-remote`, with GitHub Flavored Markdown support (`remark-gfm`) and HTML sanitization (`rehype-sanitize`) applied by default. The `generatePostMetadata` helper fetches the same post and returns a Next.js `Metadata` object populated with the post's title, teaser, keywords, and author.

Both `AtomPage` and `Atom` are async server components, which means the project key never reaches the browser. Wrapping them in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without `<Suspense>`, rendering blocks until the fetch completes and the user sees no loading state.

## Configuring Tailwind CSS

The SDK's components ship with Tailwind class names. For those classes to be included in your build output, add the SDK's component path to your Tailwind content array and register the typography plugin:

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

Without this step, the SDK components will render with missing styles.

## Customizing markdown rendering

The `Atom` component accepts optional `remarkPlugins` and `rehypePlugins` props if you need to extend the default markdown pipeline:

```tsx
<Atom
  projectKey={process.env.ATOM_PROJECT_KEY!}
  postId={params.id}
  remarkPlugins={[myCustomRemarkPlugin]}
  rehypePlugins={[myCustomRehypePlugin]}
/>
```

Your plugins are appended after the built-in defaults (`remark-gfm` and `rehype-sanitize`), so you can layer on additional behavior without replacing the base processing.

## SEO helpers

The SDK also exports two utilities for search engine optimization.

`generatePostMetadata` (shown in the blog post example above) returns a Next.js `Metadata` object with the post's title, description (from the teaser field), keywords, and author. Use it in your page's `generateMetadata` export.

`generateSitemap` produces an array of sitemap entries for all posts in a project:

```tsx
// app/sitemap.ts
import { generateSitemap } from 'atom-nextjs';

export default async function sitemap() {
  const blogRoutes = await generateSitemap(
    process.env.ATOM_PROJECT_KEY!,
    'https://yoursite.com/blog'
  );

  return [
    { url: 'https://yoursite.com', lastModified: new Date() },
    ...blogRoutes,
  ];
}
```

Each entry includes the post's URL, its `lastModified` date (from the post's `updatedAt` field), and a priority value. The blog index route is also included automatically.

## What Atom is (and isn't) responsible for

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else.

The current free plan ("Single") supports up to 2 projects, 100 posts per project, and a 10,000 character body length limit. Posts on the free plan include a small Atom watermark. Paid plans with higher limits are planned but not yet available.

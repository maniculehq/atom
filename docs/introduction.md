# Introduction

Atom is a headless CMS built for Next.js. You write and manage blog posts in the Atom dashboard, then render them in your Next.js app using the `atom-nextjs` SDK. Because the SDK uses React Server Components, your project key stays on the server and your content is fetched at render time with zero client-side JavaScript.

## How it works

1. **Create a project** in the Atom dashboard. Each project holds a collection of blog posts and has a unique project key (a string starting with `atom-` that authenticates SDK requests).

2. **Install the SDK** (`atom-nextjs`) in your Next.js app.

3. **Drop in two components** — `AtomPage` for your blog list and `Atom` for individual posts — and pass them your project key. The SDK handles fetching, Markdown rendering, and metadata generation.

That's the whole integration. Your content lives in Atom, your site stays in your repo, and the SDK connects the two at render time using server components.

## What the integration looks like

Here's a complete blog list page and a single post page — the only two files you need:

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { cookies } from 'next/headers';
import { Suspense } from 'react';

export const metadata = { title: 'Blog' };

export default function Blog() {
  const _cookies = cookies(); // opt out of caching

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

```tsx
// app/blog/[id]/page.tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { cookies } from 'next/headers';
import { Suspense } from 'react';

type BlogParams = { params: { id: string } };

export async function generateMetadata({ params }: BlogParams) {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
}

export default function BlogPost({ params }: BlogParams) {
  const _ = cookies(); // opt out of caching

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

The SDK returns an `ApiResponse` wrapper for all data fetches:

```ts
type ApiResponse<T = null> = {
  response: T;       // the data (project or post)
  success: boolean;  // whether the request succeeded
  message: string;   // error message if success is false
};
```

## SDK exports at a glance

The `atom-nextjs` package exports everything you need to build a blog:

| Export | Purpose |
|---|---|
| `AtomPage` | Server component — renders a list of post cards for a project |
| `Atom` | Server component — renders a single post with full Markdown body |
| `AtomBody` | Compiles and renders raw Markdown/MDX (uses `next-mdx-remote`) |
| `AtomPostCard` | Individual post card used inside `AtomPage` |
| `generatePostMetadata` | Generates Next.js `Metadata` for a single post (title, description, keywords, authors) |
| `generateSitemap` | Generates sitemap entries for all posts in a project |
| `getPost` | Fetches a single post by project key and post ID |
| `getProject` | Fetches a project and its posts by project key |
| `AtomLoadingSkeleton` | Loading skeleton for `AtomPage` |
| `AtomArticleSkeleton` | Loading skeleton for `Atom` |

## Prerequisites

- **Next.js 14+** with the App Router enabled (the SDK uses server components)
- **TailwindCSS** with the `@tailwindcss/typography` plugin (the SDK applies `prose` classes for Markdown styling)
- An Atom account with at least one project and its project key

> **Note:** Atom does not currently support dark mode. The rendered components assume a light-colored background.

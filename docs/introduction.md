# Atom

Atom is a headless CMS built specifically for Next.js. You write and manage your content (projects and posts in Markdown/MDX) through the Atom dashboard, then pull that content into your own Next.js site using the `atom-nextjs` npm package. No custom API calls, no data-fetching boilerplate. Just React server components that do the work for you.

## How it works

In Atom, a **project** is a named container for your blog posts. Each project gets a unique **project key** (a Bearer token used to identify which project's content to fetch). You copy that key from the dashboard and drop it into your Next.js app as an environment variable.

From there, the `atom-nextjs` SDK handles everything: fetching posts from the Atom API, compiling your MDX, and rendering the result as React server components.

## Get started in three steps

First, install the SDK:

```bash
npm install atom-nextjs
```

Then add your project key to `.env.local`:

```bash
ATOM_PROJECT_KEY=atom-your-project-key-here
```

Now you can render your blog index and individual post pages. Here's a complete example using Next.js App Router:

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function Blog() {
  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage
        projectKey={process.env.ATOM_PROJECT_KEY!}
        baseRoute="/blog"
      />
    </Suspense>
  );
}
```

```tsx
// app/blog/[id]/page.tsx
import { Atom, AtomArticleSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function BlogPost({ params }: { params: { id: string } }) {
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

`AtomPage` fetches all posts in your project and renders a linked card grid. `Atom` fetches and renders a single post: title, cover image, author, date, and the compiled MDX body. Both are async server components, so wrapping them in `Suspense` gives you a loading state while the content arrives.

Visit the Atom dashboard at [cmsatom.netlify.app](https://cmsatom.netlify.app) to create an account, set up your first project, and get your project key.

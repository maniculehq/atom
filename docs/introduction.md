# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app, with no lock-in on layout, styling, or routing.

## How the dashboard and SDK work together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts). You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components, `AtomPage` to list posts and `AtomPost` to display a single post, pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

```bash
npm install atom-nextjs
```

## Set up a blog in two files

Store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`) and reference it in each component:

```tsx
// app/blog/page.tsx — renders a linked card list of all posts in your project
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function Blog() {
  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage
        baseRoute="/blog"                              // string — URL prefix for post links (required)
        projectKey={process.env.ATOM_PROJECT_KEY!}     // string — your project's Bearer token (required)
      />
    </Suspense>
  );
}
```

`AtomPage` fetches all posts in your project and renders each as a linked card. The cards link to `{baseRoute}/{post.id}`, so a post with id `my-first-post` becomes `/blog/my-first-post`.

```tsx
// app/blog/[id]/page.tsx — fetches and renders a single post by its ID
import { AtomPost, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { Suspense } from 'react';

// Optional: generates <title> and <meta> tags from the post's title, teaser, and image
export const generateMetadata = async ({ params }: { params: { id: string } }) => {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
};

export default function BlogPage({ params }: { params: { id: string } }) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <AtomPost
        projectKey={process.env.ATOM_PROJECT_KEY!}    // string — your project's Bearer token (required)
        postId={params.id}                             // string — the post ID from the URL (required)
      />
    </Suspense>
  );
}
```

`AtomPage` and `AtomPost` are async server components that fetch from the Atom API. Wrapping them in `<Suspense>` lets Next.js stream the page immediately and show a skeleton while the fetch is in progress. Without it, rendering blocks until the fetch completes and there is no loading state.

### What the API returns

Under the hood, these components call the Atom API and receive the following shapes:

**Post list** (used by `AtomPage`):

```ts
type ClientPost = {
  id: string;          // unique post identifier, used in URLs
  title: string;
  teaser: string;      // short summary shown on cards
  author: string;
  image?: string | null;
  createdAt: Date;
  updatedAt: Date;
};
```

**Single post** (used by `AtomPost`):

```ts
type Post = {
  id: string;
  title: string;
  body: string;        // full markdown content
  teaser: string;
  author: string;
  image: string | null;
  keywords?: string[];  // optional SEO keywords
  createdAt: Date;
  updatedAt: Date;
};
```

You don't need to work with these types directly (the components handle fetching and rendering), but they're useful if you want to build custom UI on top of the API.

## What Atom owns, and what stays yours

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline (writing, storing, and serving posts) and your Next.js app owns everything else, including layout, styling, routing, and deployment.

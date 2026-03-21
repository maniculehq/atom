# Getting Started with Atom and atom-nextjs

Atom is a headless CMS for Next.js. You write and manage posts in the Atom dashboard, then pull that content into your own Next.js site using the `atom-nextjs` package. This guide walks you from account setup to a working blog in your app.

## Prerequisites

- A Next.js 14 project using the App Router
- TailwindCSS configured in your project (the SDK components use Tailwind utility classes)
- Node.js 18+

---

## How it works

Before diving into steps, here's the mental model you need:

Atom stores your content in **Projects**. A Project is a named container that holds a collection of Posts. When you create a Project, Atom generates a **project key** — a secret Bearer token that looks like `atom-<random base64 string>`. Your Next.js app presents that key when it calls the Atom API, which responds with your posts. No user session or login is required on the reader-facing side.

The `atom-nextjs` package wraps those API calls and ships ready-to-use React Server Components that render your post list and individual post pages (including MDX compilation via `next-mdx-remote`).

The flow end-to-end:

```
Atom Dashboard → you write posts
     ↓
Atom API (https://cmsatom.netlify.app/api)
     ↑
atom-nextjs SDK (your Next.js app, authenticated with your project key)
     ↓
Your readers
```

---

## Step 1: Create an Atom account and a project

1. Go to [cmsatom.netlify.app](https://cmsatom.netlify.app) and sign up for an account.
2. Once in the dashboard, click **Create project** and give it a name (e.g. "My Blog").
3. Open the project. You'll see your **project key** displayed on the project page. Copy it — you'll need it in Step 3.

The project key is the only credential your Next.js app needs. Guard it like any other API secret.

---

## Step 2: Write your first post

Inside your project, click **Create post**. Fill in:

- **Title** — displayed as the `<h1>` on the post page
- **Author** — shown below the title
- **Teaser** — a short description used in post cards and SEO metadata
- **Body** — full Markdown/MDX content

Save and publish the post. You're done with the CMS side.

---

## Step 3: Install atom-nextjs

In your Next.js project:

```bash
npm install atom-nextjs
```

Then add your project key to `.env.local`:

```bash
ATOM_PROJECT_KEY=atom-your-key-here
```

Never commit this file. Add `.env.local` to your `.gitignore` if it isn't already there.

---

## Step 4: Build the blog listing page

Create `app/blog/page.tsx`. This page fetches all posts in your project and renders them as a grid of cards.

```tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export default function BlogPage() {
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

`AtomPage` is a React Server Component. It calls the Atom API at render time, then renders an `AtomPostCard` for each post. The `baseRoute` prop tells each card what URL prefix to link to — here, a post with id `abc123` will link to `/blog/abc123`.

Wrap it in `<Suspense>` with `<AtomLoadingSkeleton />` as the fallback so readers see a skeleton UI while the API call resolves instead of a blank page.

---

## Step 5: Build the individual post page

Create `app/blog/[id]/page.tsx`. This page fetches a single post by its ID and renders the full article, including compiled MDX.

```tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { Suspense } from 'react';

type Props = { params: { id: string } };

export const generateMetadata = async ({ params }: Props) => {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
};

export default function PostPage({ params }: Props) {
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

`generateMetadata` uses `generatePostMetadata` to populate the page's `<title>`, `description`, `keywords`, and `authors` fields from the post data — no extra fetch needed on your end.

`Atom` renders the full article: the post title, an optional hero image, the author name, the publish date, and the compiled MDX body. It uses `@tailwindcss/typography` prose classes, so your Tailwind config should include the typography plugin for the best rendering. If you haven't added it yet:

```bash
npm install @tailwindcss/typography
```

```js
// tailwind.config.ts
plugins: [require('@tailwindcss/typography')],
```

---

## Step 6: Add a sitemap (optional but recommended)

`atom-nextjs` exports `generateSitemap`, which builds sitemap entries for every post in your project. Drop this in `app/sitemap.ts`:

```ts
import { MetadataRoute } from 'next';
import { generateSitemap } from 'atom-nextjs';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const postRoutes = await generateSitemap(
    process.env.ATOM_PROJECT_KEY!,
    'https://yourdomain.com/blog'
  );

  return [
    {
      url: 'https://yourdomain.com',
      lastModified: new Date(),
      priority: 0.7,
    },
    ...postRoutes,
  ];
}
```

Each post entry includes the post URL, its `lastModified` date, and a default `priority` of `0.5`. The blog index route (`/blog`) is included automatically at `priority: 0.6`.

---

## What you should see

Run `npm run dev` and open `http://localhost:3000/blog`. You should see a grid of post cards, each showing the post image (if set), title, teaser, author, and date. Clicking a card takes you to the full article at `/blog/[post-id]`.

If a post isn't showing up, double-check that:
- Your `ATOM_PROJECT_KEY` in `.env.local` matches the key shown in your Atom dashboard project.
- The post is published (not in draft state) in the dashboard.
- You restarted `next dev` after editing `.env.local`.

---

## Next steps

- **Pass remark/rehype plugins to `Atom`** — Both `Atom` and `AtomBody` accept `remarkPlugins` and `rehypePlugins` arrays if you want to extend MDX rendering beyond the defaults (`remark-gfm` and `rehype-sanitize`).
- **Use the raw data fetchers** — `getPost(projectKey, postId)` and `getProject(projectKey)` return typed API responses you can use to build completely custom UI instead of the built-in components.
- **Manage multiple blogs** — Each Atom project gets its own key, so you can run multiple independent blogs from one account and connect each to a different Next.js app.

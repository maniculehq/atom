# Quickstart

Get a working blog inside your Next.js app in under ten minutes, pulling content from the Atom dashboard.

## Prerequisites

- A Next.js 14 project using the App Router
- Node.js and npm (or your preferred package manager)
- An Atom account (free at [cmsatom.netlify.app](https://cmsatom.netlify.app))

---

## 1. Create an account and set up a project

Go to [cmsatom.netlify.app/signup](https://cmsatom.netlify.app/signup) and create a free account. After signing in you land on the Projects page.

Click **Create project** and give it a name. Your project is a container for all your blog posts. Once created, open it and you will see your **project key** in the sidebar. It looks like this:

```
atom-abc123xyz...
```

Copy it. You will need it in the next step.

## 2. Write your first post

Inside your new project, click **Create post**. Fill in the title, author, and teaser, then write the body using the MDX editor. When you save, the post is immediately available via the Atom API. The post ID is generated automatically; you will use it later to link directly to individual posts.

## 3. Install the SDK in your Next.js app

In your own Next.js project, install `atom-nextjs`:

```bash
npm install atom-nextjs
```

Then add your project key as an environment variable. Create or update your `.env.local` file:

```bash
ATOM_PROJECT_KEY=atom-abc123xyz...
```

This variable is only read on the server, so the key is never sent to the browser.

## 4. Add a blog listing page

Create `app/blog/page.tsx`. The `AtomPage` component fetches all posts in your project and renders them as a card grid. Pass it your project key and the base URL of your blog so the cards link to the right place.

```tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';
import { cookies } from 'next/headers';

export default function BlogPage() {
  // Reading cookies opts this route out of Next.js caching,
  // so visitors always see your latest posts.
  const _cookies = cookies();

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

The `Suspense` boundary shows a skeleton placeholder while the posts load. `AtomLoadingSkeleton` is included in the SDK for exactly this purpose, but you can swap in any fallback you like.

## 5. Add an individual post page

Create `app/blog/[id]/page.tsx`. The `Atom` component fetches and renders a single post, including the title, cover image, author, date, and the full MDX body.

```tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { Suspense } from 'react';
import { cookies } from 'next/headers';

type Props = { params: { id: string } };

export async function generateMetadata({ params }: Props) {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
}

export default function PostPage({ params }: Props) {
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

`generatePostMetadata` automatically populates `<title>`, `description`, `keywords`, and `authors` from the post data in Atom, so your `<head>` stays in sync with your content without any extra work.

## 6. (Optional) Keep your sitemap up to date automatically

Create `app/sitemap.ts` and use `generateSitemap` to produce sitemap entries for every post in your project. Combine it with your own static routes:

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

Each entry includes the post's `lastModified` date and a default priority of `0.5`. The blog index route is also included at priority `0.6`. Whenever you publish or edit a post in the Atom dashboard, the next sitemap request picks up the change automatically.

## Verify it works

Start your dev server:

```bash
npm run dev
```

Open `http://localhost:3000/blog`. You should see your post listed as a card. Click it and the full article renders at `http://localhost:3000/blog/<post-id>`. If the post body contains MDX (components, callouts, etc.), it will be compiled and rendered on the server.

If the page loads but no posts appear, double-check that `ATOM_PROJECT_KEY` in `.env.local` matches the key shown in your Atom dashboard project, and that the project contains at least one saved post.

## What's next

- **Call the API directly**: `getProject(projectKey)` and `getPost(projectKey, postId)` return raw data if you want to build a fully custom layout instead of using the built-in components.
- **Custom MDX rendering**: `Atom` accepts `remarkPlugins` and `rehypePlugins` props if you need to extend the MDX pipeline.
- **AtomBody for custom layouts**: import `AtomBody` directly and pass it a `body` string to render MDX inside your own article template.

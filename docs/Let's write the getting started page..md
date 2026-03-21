# Getting Started with Atom and atom-nextjs

Atom is a headless CMS for Next.js. You write and manage posts in the Atom dashboard, then pull that content into your own Next.js site using the `atom-nextjs` package. This guide walks you from account setup to a working blog in your app.

## Prerequisites

- **Next.js 14** with the App Router — the SDK components are React Server Components and require the App Router's server-side rendering model.
- **TailwindCSS** configured in your project — the SDK ships pre-styled components that rely on Tailwind utility classes, including `@tailwindcss/typography` for post body rendering.
- **Node.js 18+**

---

## Understand the mental model before writing any code

Atom stores your content in **Projects**. A Project is a named container that holds a collection of Posts. When you create a Project, Atom generates a **project key** — a secret Bearer token that looks like `atom-<random base64 string>`. Your Next.js app presents that key when it calls the Atom API, which responds with your posts. No user session or login is required on the reader-facing side.

The `atom-nextjs` package wraps those API calls and ships ready-to-use React Server Components that render your post list and individual post pages (including MDX compilation via `next-mdx-remote`).

The flow end-to-end:

```
Atom Dashboard → you write posts
     ↓
Atom API (https://cmsatom.netlify.app/api)
     ↑  your project key authenticates every request
atom-nextjs SDK (your Next.js app)
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

- **Title** — displayed as the `<h1>` on the post page and used in SEO metadata.
- **Author** — shown below the title on the article page.
- **Teaser** — a short description used in post cards and as the SEO `description` tag.
- **Body** — full Markdown/MDX content compiled at render time by `next-mdx-remote`.

Save and publish the post. You're done with the CMS side.

---

## Step 3: Install atom-nextjs and add your project key

```bash
npm install atom-nextjs @tailwindcss/typography
```

Then add your project key to `.env.local`:

```bash
ATOM_PROJECT_KEY=atom-your-key-here
```

Never commit this file. Add `.env.local` to your `.gitignore` if it isn't already there.

Also update your Tailwind config so that both the SDK's component files and the typography plugin are included — without this, Tailwind will purge the SDK's styles in production:

```ts
// tailwind.config.ts
module.exports = {
  content: [
    // ... your existing paths
    './node_modules/atom-nextjs/src/components/*.{ts,tsx}',
  ],
  plugins: [require('@tailwindcss/typography')],
};
```

---

## Step 4: Build the blog listing page

Create `app/blog/page.tsx`. This page fetches all posts in your project and renders them as a grid of cards.

```tsx
// app/blog/page.tsx
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

`AtomPage` is a React Server Component. It calls the Atom API at render time, then renders an `AtomPostCard` for each post in your project.

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `projectKey` | `string` | Yes | Your Atom project key from `.env.local`. |
| `baseRoute` | `string` | Yes | URL prefix for post links. A post with id `abc123` and `baseRoute="/blog"` links to `/blog/abc123`. |
| `title` | `boolean` | No (default: `true`) | Whether to render the project name as an `<h1>` above the post grid. |

Wrap it in `<Suspense>` with `<AtomLoadingSkeleton />` as the fallback so readers see a skeleton UI while the API call resolves instead of a blank page.

---

## Step 5: Build the individual post page

Create `app/blog/[id]/page.tsx`. This page fetches a single post by its ID and renders the full article, including compiled MDX.

```tsx
// app/blog/[id]/page.tsx
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

`Atom` renders the full article: the post title, an optional hero image, the author name, the publish date, and the compiled MDX body. It uses `@tailwindcss/typography` prose classes for the body, which is why the Tailwind config update in Step 3 matters.

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `projectKey` | `string` | Yes | Your Atom project key. |
| `postId` | `string` | Yes | The post ID from the URL parameter. |
| `remarkPlugins` | `any[]` | No | Additional remark plugins passed to `next-mdx-remote`. |
| `rehypePlugins` | `any[]` | No | Additional rehype plugins passed to `next-mdx-remote`. |

---

## Step 6: Add a sitemap (optional but recommended)

`atom-nextjs` exports `generateSitemap`, which builds sitemap entries for every post in your project. Drop this in `app/sitemap.ts`:

```ts
// app/sitemap.ts
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

`generateSitemap` returns an array of sitemap entries. Each post gets its own entry at `priority: 0.5`, and the blog index route (`/blog`) is included automatically at `priority: 0.6`:

```ts
// Shape of each entry returned by generateSitemap
[
  { url: 'https://yourdomain.com/blog/abc123', lastModified: Date, priority: 0.5 },
  // ... one per published post
  { url: 'https://yourdomain.com/blog', lastModified: Date, priority: 0.6 },
]
```

---

## Verify everything is working

Run `npm run dev` and open `http://localhost:3000/blog`. You should see a grid of post cards, each showing the post image (if set), title, teaser, author, and date. Clicking a card takes you to the full article at `/blog/[post-id]`.

If a post isn't showing up, check these in order:

1. Your `ATOM_PROJECT_KEY` in `.env.local` matches the key shown in your Atom dashboard project.
2. The post is published (not in draft state) in the dashboard.
3. You restarted `next dev` after editing `.env.local` — Next.js only reads env files on startup.

---

## Next steps

**Extend MDX rendering** — Both `Atom` and `AtomBody` accept `remarkPlugins` and `rehypePlugins` arrays if you want to go beyond the defaults (`remark-gfm` and `rehype-sanitize`). Pass any compatible plugin directly, for example `remarkPlugins={[remarkMath]}`.

**Build a fully custom UI** — If the built-in components don't match your design, use the raw data fetchers instead. `getPost(projectKey, postId)` returns a typed `ApiResponse<Post>` and `getProject(projectKey)` returns `ApiResponse<ClientProject>`, giving you direct access to all post fields to render however you like.

**Run multiple blogs** — Each Atom project gets its own key, so you can run multiple independent blogs from one account and connect each to a different Next.js app (or a different route in the same app).

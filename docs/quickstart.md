# Quickstart

Go from zero to a working blog in your Next.js app, with content managed through the Atom dashboard. By the end of this guide, you'll have a blog listing page and individual post pages powered by the Atom CMS.

Here's what the core integration looks like — a single server component that fetches and renders your blog:

```tsx
// app/blog/page.tsx
import { AtomPage } from 'atom-nextjs';

export default function Blog() {
  return <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />;
}
```

The steps below walk you through setting up your Atom project, installing the SDK, and building out the full pages with loading states and SEO metadata.

## Prerequisites

- A [Next.js 13+](https://nextjs.org/) project using the App Router
- [Tailwind CSS](https://tailwindcss.com/) configured in your project
- Node.js 18 or later

## 1. Create an Atom account

Head to [cmsatom.netlify.app/signup](https://cmsatom.netlify.app/signup) and sign up with your email and password. Once registered, you'll land on the dashboard's **Projects** page.

## 2. Create a project and grab your project key

Click **Create Project**, give it a name (for example, "My Blog"), and confirm. The project opens in the editor view.

In the left sidebar, click **Copy project key**. This key is how the SDK authenticates requests to the Atom API — it's sent as a Bearer token in every request the SDK makes on your behalf. You'll need it in the next step.

## 3. Write your first post

While still in the project editor, click **Create new post** at the bottom of the sidebar. Fill in the title, author name, teaser (a short summary shown on post cards), and body content using markdown. Add a cover image URL and comma-separated keywords. Click **Create post** to save.

You now have content ready to fetch. Time to wire it up in your Next.js app.

## 4. Install the SDK

In your Next.js project, install the `atom-nextjs` package and the Tailwind typography plugin (used for rendering markdown content):

```bash
npm install atom-nextjs @tailwindcss/typography
```

## 5. Store your project key

Add your project key to `.env.local` so it stays out of source control:

```env
ATOM_PROJECT_KEY=atom-xxxxxxxxxxxx
```

Replace `atom-xxxxxxxxxxxx` with the key you copied in step 2.

Because the SDK components are server components, this variable is only read on the server. It never reaches the browser.

## 6. Configure Tailwind to include SDK styles

The SDK ships its own components that use Tailwind classes. You need to tell Tailwind to scan the SDK's source files and enable the typography plugin so that those classes end up in your final CSS.

Update your `tailwind.config.ts` (or `.js`):

```ts
// tailwind.config.ts

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    // your existing paths...
    './node_modules/atom-nextjs/src/components/*.{ts,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [
    // your existing plugins...
    require('@tailwindcss/typography'),
  ],
};
```

The `content` entry ensures Tailwind includes the SDK's component classes in your build. The `@tailwindcss/typography` plugin provides the `prose` styles that format the rendered markdown.

## 7. Create the blog listing page

Create `app/blog/page.tsx`. The `AtomPage` component fetches all posts in your project and renders them as linked cards:

```tsx
// app/blog/page.tsx

import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';

export const metadata = {
  title: 'Blog',
};

export default function Blog() {
  return (
    <Suspense fallback={<AtomLoadingSkeleton />}>
      <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
    </Suspense>
  );
}
```

`AtomPage` is an async server component. It calls the Atom API with your project key as a Bearer token, retrieves the project's posts, and renders a card for each one. The `baseRoute` prop tells it where to link individual posts (each card links to `/blog/{post-id}`).

Wrapping it in `<Suspense>` lets Next.js stream the page shell immediately and show `AtomLoadingSkeleton` while the fetch completes.

### `AtomPage` props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | — | Your Atom project key (required) |
| `baseRoute` | `string` | — | URL prefix for post links, e.g. `"/blog"` (required) |
| `title` | `boolean` | `true` | Whether to render the project title as an `<h1>`. Set to `false` if you render your own heading. |

## 8. Create the single post page

Create `app/blog/[id]/page.tsx`. The `Atom` component fetches and renders one post, including its full markdown body:

```tsx
// app/blog/[id]/page.tsx

import { Atom, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { Suspense } from 'react';

type BlogParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogParams) => {
  return await generatePostMetadata(
    process.env.ATOM_PROJECT_KEY!,
    params.id
  );
};

export default function BlogPage({ params }: BlogParams) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

`generatePostMetadata` fetches the post and returns a Next.js `Metadata` object with the post's title, description (teaser), keywords, and author. This means your blog pages get proper `<title>` and `<meta>` tags with no extra work.

The `Atom` component fetches the post content, compiles the markdown body to HTML (using MDX with `remark-gfm` for GitHub-flavored markdown), and renders the full article with title, author, date, cover image, and body.

### `Atom` props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | — | Your Atom project key (required) |
| `postId` | `string` | — | The post ID from the URL params (required) |
| `remarkPlugins` | `any[]` | `[]` | Additional [remark plugins](https://github.com/remarkjs/remark/blob/main/doc/plugins.md) to apply during markdown compilation |
| `rehypePlugins` | `any[]` | `[]` | Additional [rehype plugins](https://github.com/rehypejs/rehype/blob/main/doc/plugins.md) to apply during HTML processing |

## 9. Verify it works

Start your dev server:

```bash
npm run dev
```

Open `http://localhost:3000/blog`. You should see a card for the post you created in step 3. Click the card to open the full post with your rendered markdown content.

If you see an error instead, double-check that:

- Your `ATOM_PROJECT_KEY` in `.env.local` matches the key from the dashboard
- You restarted the dev server after adding the environment variable
- Your Tailwind config includes the SDK content path

## Next steps

- **Add more posts** in the Atom dashboard. They appear on your blog automatically — no redeployment needed, since the SDK fetches content at request time.
- **Wrap components in your own layout** by nesting `AtomPage` and `Atom` inside your app's container components.
- **Generate a sitemap** using the `generateSitemap` helper from the SDK for better SEO:

  ```tsx
  // app/sitemap.ts

  import { generateSitemap } from 'atom-nextjs';

  export default async function sitemap() {
    return await generateSitemap(
      process.env.ATOM_PROJECT_KEY!,
      'https://yourdomain.com/blog'
    );
  }
  // Returns: [{ url: string, lastModified: Date, priority: number }, ...]
  ```

- **Hide the project title** by passing `title={false}` to `AtomPage` if you want to render your own heading.

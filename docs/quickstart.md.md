# Quickstart

Go from zero to a working blog in your Next.js app in under ten minutes.

## Prerequisites

- [Node.js](https://nodejs.org/) 16 or later
- A Next.js 14 project using the App Router
- [TailwindCSS](https://tailwindcss.com/) installed in your project
- An Atom account (free) at [cmsatom.netlify.app](https://cmsatom.netlify.app)

## 1. Create a project in the Atom dashboard

Sign up or sign in at [cmsatom.netlify.app](https://cmsatom.netlify.app/signup). Once you're in the dashboard, click **Create project** and give it a name (for example, "My Blog").

After the project is created, open it and click **Copy project key** in the bottom-left sidebar. This key is a Bearer token that authenticates your Next.js app against the Atom API. You'll need it in the next step.

## 2. Write a test post

While you're still in the project, click **Create post**. Fill in the fields:

- **Title**: "Hello World"
- **Author**: your name
- **Body**: some markdown content (GitHub Flavored Markdown is supported)
- **Teaser**: a short summary that appears on the blog listing page
- **Keywords**: comma-separated tags (optional)
- **Cover image link**: a URL to an image (optional)

Save the post. You now have content to display.

## 3. Store your project key

Back in your Next.js app, create a `.env.local` file in the project root (if you don't already have one) and add your project key:

```bash
ATOM_PROJECT_KEY=atom-xxxxxxxxxxxxxxxx
```

Replace `atom-xxxxxxxxxxxxxxxx` with the key you copied from the dashboard. Because the SDK components are server components, this key stays on the server and never reaches the browser.

## 4. Install the SDK and typography plugin

Install `atom-nextjs` and the Tailwind typography plugin (used by the post renderer for proper article styling):

```bash
npm install atom-nextjs @tailwindcss/typography
```

## 5. Configure Tailwind CSS

The SDK ships its own components with Tailwind classes, so your Tailwind config needs to scan the package's source files. It also uses the `@tailwindcss/typography` plugin for the `prose` classes that style rendered markdown.

Open your `tailwind.config.ts` (or `.js`) and add both:

```ts
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './app/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './node_modules/atom-nextjs/src/components/*.{ts,tsx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [require('@tailwindcss/typography')],
};
```

The important additions are the `./node_modules/atom-nextjs/src/components/*.{ts,tsx}` content path and the `@tailwindcss/typography` plugin.

## 6. Create the blog listing page

Create `app/blog/page.tsx`. This page fetches all posts from your project and renders them as linked cards:

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';
import { Metadata } from 'next';

export const metadata: Metadata = {
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

`AtomPage` is an async server component that fetches your project's posts from the Atom API and renders each one as a card. The `baseRoute` prop tells it where individual posts live, so each card links to `/blog/[id]`. Wrapping it in `<Suspense>` lets Next.js stream the page shell immediately and show a skeleton while the data loads.

## 7. Create the single post page

Create `app/blog/[id]/page.tsx`. This page renders one post and generates its metadata (title, description, Open Graph tags) automatically:

```tsx
// app/blog/[id]/page.tsx
import { AtomPost, AtomArticleSkeleton, generatePostMetadata } from 'atom-nextjs';
import { Suspense } from 'react';

type BlogParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogParams) => {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
};

export default function BlogPage({ params }: BlogParams) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <AtomPost projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

`AtomPost` fetches a single post by ID and renders it as a full article with a title, cover image, author, date, and the markdown body (compiled with `remark-gfm` for GitHub Flavored Markdown). The `generatePostMetadata` function makes a separate request to build a Next.js `Metadata` object, so your posts get proper `<title>` and `<meta>` tags without any extra work.

## 8. Start the dev server and verify

Run your development server:

```bash
npm run dev
```

Open [http://localhost:3000/blog](http://localhost:3000/blog). You should see a card for the "Hello World" post you created earlier. Click the card to navigate to the full post.

If you see the project title, the post card, and the rendered markdown content, everything is working.

## Add more demo content

Head back to the Atom dashboard and create a few more posts to fill out your blog. Each post you create will appear on the listing page the next time it loads (no redeploy needed since `AtomPage` fetches at request time).

Here are some ideas for demo posts to test different formatting:

- A post with a cover image to verify image rendering
- A post with code blocks, tables, or lists to see `remark-gfm` in action
- A post with a longer body to test the article layout

## Optional: wrap components in your own layout

The SDK components render content without any surrounding layout. If you have a header, footer, or container component, wrap the SDK output inside it:

```tsx
// app/blog/page.tsx
import { AtomPage, AtomLoadingSkeleton } from 'atom-nextjs';
import { Suspense } from 'react';
import { MyLayout } from '@/components/MyLayout';

export default function Blog() {
  return (
    <MyLayout>
      <Suspense fallback={<AtomLoadingSkeleton />}>
        <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
      </Suspense>
    </MyLayout>
  );
}
```

The same pattern works for the single post page. The SDK stays inside whatever structure you provide.

## What you just built

With two files and one environment variable, your Next.js app now has:

- A blog listing page that shows all posts as cards
- Individual post pages with full markdown rendering and SEO metadata
- Loading skeletons that display while content streams in
- Server-side rendering that keeps your API key off the client

## Next steps

- **[Introduction](introduction.md)**: Learn how the dashboard and SDK fit together, and see the full list of components and functions.
- **Add a sitemap**: Use the `generateSitemap` function to create `app/sitemap.ts` and give search engines a map of your blog posts.
- **Customize rendering**: Pass custom `remarkPlugins` or `rehypePlugins` to `AtomPost` to extend the markdown pipeline.

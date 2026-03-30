# Introduction to Atom

Atom is a headless CMS built specifically for Next.js. You write and manage blog posts in the Atom dashboard, then render them in your own Next.js site using the `atom-nextjs` SDK, a set of async React server components that fetch and display your content with zero client-side JavaScript.

The system has two parts:

1. **The Dashboard** is where you create an account, organize posts into projects, and write content in a Markdown editor. Each project gets a unique API key (called a `project_key`) that your Next.js app uses to pull content.

2. **The `atom-nextjs` SDK** is an npm package you install in your Next.js app. It provides server components like `Atom` and `AtomPage` that fetch your posts at render time and output styled HTML. No API calls to write, no state to manage, no loading spinners to build.

## How it works

When you create a project in the Atom dashboard, a `project_key` is generated automatically. This key is a base64 Bearer token that identifies your project.

The SDK components use this key behind the scenes. When Next.js renders a page that includes an `Atom` or `AtomPage` component, the component makes a server-side fetch to the Atom API, retrieves your posts, converts the Markdown to HTML (using `next-mdx-remote` with GitHub Flavored Markdown and sanitized output), and returns fully rendered content.

Because these are async server components, the data fetching happens on the server during rendering. Your visitors never make API calls from their browsers, and your `project_key` stays on the server where it belongs.

## Getting started with the dashboard

1. Sign up at the Atom dashboard and create an account.
2. Create a new project. Give it a name (up to 50 characters) that describes your blog or content collection.
3. Copy the `project_key` from your project. You'll need this for the SDK.
4. Write your first post using the built-in Markdown editor. Each post has a title, author, teaser (short description), optional cover image, optional keywords, and a Markdown body.

That's it on the dashboard side. Your content is now available through the API.

## Adding the SDK to your Next.js app

Install the package:

```bash
npm i atom-nextjs@latest
```

Store your project key as an environment variable. In your `.env.local` file:

```
ATOM_PROJECT_KEY=your_project_key_here
```

### Displaying a list of posts

The `AtomPage` component fetches all posts from a project and renders them as linked cards. Here's a minimal blog listing page:

```tsx
import { AtomPage, AtomLoadingSkeleton } from "atom-nextjs";
import { Suspense } from "react";

export default function Blog() {
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

`AtomPage` takes two required props: `projectKey` (your API key) and `baseRoute` (the URL path where individual posts live). It also accepts an optional `title` prop (defaults to `true`) that controls whether the project title is shown above the post list.

Each post card displays the title, author, date, teaser, and cover image. Clicking a card navigates to `{baseRoute}/{post.id}`.

The `AtomLoadingSkeleton` component provides a placeholder layout while the server component resolves, so your page has a smooth loading state.

### Displaying a single post

The `Atom` component renders a full blog post, including the title, cover image, author, publish date, and the Markdown body:

```tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from "atom-nextjs";
import { Suspense } from "react";

type BlogParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogParams) => {
  return await generatePostMetadata(
    process.env.ATOM_PROJECT_KEY!,
    params.id
  );
};

export default function BlogPost({ params }: BlogParams) {
  return (
    <Suspense fallback={<AtomArticleSkeleton />}>
      <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </Suspense>
  );
}
```

A few things to note here. The `generatePostMetadata` function fetches the post and returns a Next.js `Metadata` object with the title, description (from the teaser), keywords, and author. This gives you SEO metadata without writing any extra code.

The `Atom` component renders the post inside an `<article>` with Tailwind Typography's `prose` classes, so the Markdown output is well-formatted by default. It also accepts optional `remarkPlugins` and `rehypePlugins` props if you need to customize Markdown processing beyond the defaults (GitHub Flavored Markdown and HTML sanitization).

`AtomArticleSkeleton` works like `AtomLoadingSkeleton` but is shaped for a single article layout instead of a card grid.

### Generating a sitemap

The SDK includes a `generateSitemap` helper that creates sitemap entries for all your posts:

```tsx
import { generateSitemap } from "atom-nextjs";

export default async function sitemap() {
  const routes = await generateSitemap(
    process.env.ATOM_PROJECT_KEY!,
    "https://yoursite.com/blog"
  );

  return routes;
}
```

This returns an array of objects with `url`, `lastModified`, and `priority` fields, matching the format Next.js expects from a `sitemap.ts` file.

### Lower-level utilities

If you need more control over rendering, the SDK exports two additional tools:

- `getProject(projectKey)` fetches all posts for a project and returns the raw API response. Useful when you want to build a completely custom listing page.
- `getPost(projectKey, postId)` fetches a single post. Useful when you want to handle the layout yourself instead of using the `Atom` component.

Both return an `ApiResponse<T>` object with three fields: `success` (boolean), `message` (string), and `response` (the data, either a `ClientProject` or a `Post`).

You can also use `AtomBody` on its own to render just the Markdown body of a post, without the surrounding title/author/image layout:

```tsx
import { AtomBody } from "atom-nextjs";

// Inside an async server component
<AtomBody body={post.body} className="space-y-6" />
```

## What you get on the free plan

The free "Single" plan includes 2 projects, up to 100 posts total, and a 10,000-character limit on post bodies. Paid plans with higher limits are coming soon.

## Rate limiting

The Atom API applies a rate limit of 30 requests per minute using a sliding window. This applies to all API calls, including those made by the SDK components during server rendering. For most blogs, this is more than enough. If you're building something with very high traffic, keep in mind that Next.js caching (ISR or static generation) will reduce the number of API calls your site makes.

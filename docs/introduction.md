# Introduction to Atom

Atom is a headless CMS built specifically for Next.js developers. You write and manage blog posts inside the Atom dashboard, and a small NPM package (`atom-nextjs`) pulls that content into your own Next.js app at render time. Your site, your design — Atom just handles the content.

## The two-part model

Atom has two distinct pieces that work together:

**The Atom dashboard** is a hosted Next.js application where you create an account, set up projects, and write posts. Each post has a title, author, teaser, optional cover image, and a Markdown body. The editor is a full Markdown editor, and the content is stored in MongoDB under the hood.

**The `atom-nextjs` SDK** is the package your blog installs. It authenticates against the Atom API using a *project key* — a secret token generated for each project — and renders your posts as React Server Components using `next-mdx-remote`. You never expose your session credentials; only the project key travels with the SDK.

## How a project key connects everything

When you create a Project in the Atom dashboard, Atom generates a unique key that looks like:

```
atom-A3kF9...
```

You store that key as an environment variable in your Next.js app (`ATOM_PROJECT_KEY`), then pass it to the SDK components. The SDK sends it as a Bearer token to the Atom API, which returns only the posts that belong to that project. No login session, no cookies — just the key.

## What you add to your blog

Getting a blog running takes two files. First, a listing page that shows all your posts:

```tsx
// app/blog/page.tsx
import { AtomPage } from 'atom-nextjs';

export default function Blog() {
  return (
    <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
  );
}
```

`AtomPage` fetches your project's posts from the Atom API and renders a grid of cards. `baseRoute` tells each card where to link — in this case, `/blog/<post-id>`.

Then, the individual post page:

```tsx
// app/blog/[id]/page.tsx
import { Atom, generatePostMetadata } from 'atom-nextjs';

export const generateMetadata = async ({ params }) => {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
};

export default async function BlogPage({ params }) {
  return (
    <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
  );
}
```

`Atom` fetches a single post and renders the title, author, publish date, cover image, and the Markdown body — already compiled to HTML via `next-mdx-remote`. `generatePostMetadata` builds a proper Next.js `Metadata` object (title, description, keywords) from the post data so you get decent SEO out of the box.

Both components are React Server Components, so no content leaks to the client and the project key stays server-side.

## The request path

When a visitor loads a blog post, here is what happens:

1. Your Next.js app renders the `Atom` Server Component.
2. The SDK calls `GET /api/posts/get/single?post_id=<id>` on the Atom API, passing `Authorization: Bearer <project_key>`.
3. Atom verifies the key, retrieves the post from MongoDB, and returns it as JSON.
4. The SDK compiles the Markdown body and returns the finished HTML to your page.

For dashboard users managing their content, there is a parallel path: the Atom dashboard authenticates via a Lucia session cookie, and every write operation (create, update, delete post or project) goes through a protected API route that validates that session before touching the database.

## What to read next

If you want to get your blog up and running, head to the **Quickstart** guide — it walks you through installing the SDK, creating your first project, and wiring up the two page routes above. If you want to understand how authentication and project keys work in depth, see **Authentication**.

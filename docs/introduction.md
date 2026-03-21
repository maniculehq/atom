# Introduction to Atom

Atom is a headless CMS built for Next.js developers. You write and manage blog posts inside the Atom dashboard, and the `atom-nextjs` npm package pulls that content into your own Next.js app at render time. Your site, your design — Atom handles the content.

## How Atom's two parts work together

Atom has two distinct pieces: a hosted dashboard where you manage content, and an SDK you install in your blog.

**The Atom dashboard** is a hosted Next.js application where you create an account, set up projects, and write posts. Each post has a title, author, teaser, optional cover image, and a Markdown body. The editor is a full Markdown editor, and the content is stored in MongoDB under the hood.

**The `atom-nextjs` SDK** is the package your blog installs. It authenticates against the Atom API using a *project key* — a secret token generated for each project — and renders your posts as React Server Components using `next-mdx-remote`. You never expose your session credentials; only the project key travels with the SDK.

## How a project key connects everything

When you create a project in the Atom dashboard, Atom generates a unique key. The key is `"atom-"` followed by 44 base64-encoded characters, for example:

```text
atom-bHutpCKwbU9wRsPuzfEnqKbmQCRQY+NxYQEZkUvV65E=
```

You store that key as an environment variable in your Next.js app, then pass it to the SDK components:

```dotenv
# .env.local
ATOM_PROJECT_KEY=atom-bHutpCKwbU9wRsPuzfEnqKbmQCRQY+NxYQEZkUvV65E=
```

The SDK sends it as a Bearer token to the Atom API, which returns only the posts that belong to that project. No login session, no cookies — only the key.

> **Free plan note:** On the free Single plan, the API appends a Markdown watermark (`This post was created using [Atom]`) to every post body before returning it. Paid plans remove the watermark.

## What you add to your blog

Getting a blog running requires two files and one install. Because the `Atom` and `AtomPage` components use Tailwind Typography `prose` classes for rendering, you must install both packages:

```bash
npm install atom-nextjs@latest @tailwindcss/typography
```

**Listing page** — shows all your posts as a grid of cards:

```tsx
// app/blog/page.tsx
import { AtomPage } from 'atom-nextjs';

export default function Blog() {
  return (
    <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
  );
}
```

`AtomPage` fetches your project's posts from the Atom API and renders a grid of cards. `baseRoute` tells each card where to link — in this case `/blog/<post-id>`. An optional `title` prop (boolean, default `true`) controls whether the project title `<h1>` is rendered above the grid.

**Individual post page** — fetches and renders a single post with SEO metadata:

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

`Atom` fetches a single post and renders the title, author, publish date, cover image, and the Markdown body compiled to HTML via `next-mdx-remote`. `generatePostMetadata` builds a Next.js `Metadata` object (title, description, keywords) from the post data.

Both components are React Server Components, so no content leaks to the client and the project key stays server-side.

## The request path

When a visitor loads a blog post, here is what happens:

1. Your Next.js app renders the `Atom` Server Component.
2. The SDK calls `GET /api/posts/get/single?post_id=<id>` on the Atom API, passing `Authorization: Bearer <project_key>`.
3. Atom verifies the key, retrieves the post from MongoDB, and returns it as JSON.
4. The SDK compiles the Markdown body and returns the finished HTML to your page.

When you manage content in the dashboard, a parallel path applies: the Atom dashboard authenticates via a Lucia session cookie, and every write operation (create, update, delete post or project) goes through a protected API route that validates that session before touching the database.

## What to read next

- **Quickstart** — install the SDK, create your first project, and wire up both page routes.
- **Authentication** — a deeper look at how project keys and session auth each work.

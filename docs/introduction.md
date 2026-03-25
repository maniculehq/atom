# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app — with no lock-in on layout, styling, or routing.

## How the two parts fit together

The system has two parts that communicate through an API key.

**The Atom dashboard** is where you create a *project* — a named collection of posts. You write content in a markdown editor, publish it, and the dashboard issues a unique `project_key` for that project. This key acts as a Bearer token (an `Authorization` header value) that authenticates your app's requests to the Atom API.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components — `AtomPage` to list posts and `Atom` to display a single post — pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

```bash
npm install atom-nextjs
```

From there, a blog route is two files. Store your project key in an environment variable (e.g. `ATOM_PROJECT_KEY` in `.env.local`) and reference it in each component:

```tsx
// app/blog/page.tsx — renders a linked card list of all posts in your project
import { AtomPage } from 'atom-nextjs';

export default function Blog() {
  return <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />;
}
```

```tsx
// app/blog/[id]/page.tsx — fetches and renders a single post by its ID
import { Atom } from 'atom-nextjs';

export default async function BlogPage({ params }: { params: { id: string } }) {
  return <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />;
}
```

Your site stays in full control of layout, styling, and routing. Atom handles the content.

## What Atom is — and isn't — responsible for

Atom is focused purely on managing and delivering blog content. It does not host your site, inject scripts into your pages, or provide a visual editor for your app's UI. This boundary is intentional: Atom owns the content pipeline, and your Next.js app owns everything else.

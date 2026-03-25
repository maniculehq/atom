# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage your blog posts in the Atom dashboard, and a small companion package (`atom-nextjs`) renders them inside your own Next.js app.

## How it works

The system has two parts that talk to each other through an API key.

**The Atom dashboard** is where you create a *project* (a named collection of posts), write content in a markdown editor, and publish. Each project gets a unique `project_key` that acts as a Bearer token for your API access.

**The `atom-nextjs` SDK** is what you install in your own site. Drop in two server components (`AtomPage` to list posts and `Atom` to display a single post), pass them your project key, and they fetch and render your content directly from the Atom API at build or request time.

```bash
npm install atom-nextjs
```

From there, a blog route is two files:

```tsx
// app/blog/page.tsx
import { AtomPage } from 'atom-nextjs';

export default function Blog() {
  return <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />;
}
```

```tsx
// app/blog/[id]/page.tsx
import { Atom } from 'atom-nextjs';

export default async function BlogPage({ params }: { params: { id: string } }) {
  return <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />;
}
```

Your site stays in full control of layout, styling, and routing. Atom handles the content.

## What Atom is not

Atom does not host your site and does not inject any scripts into your pages. It is not a page builder or a visual editor for your app's UI. It is focused purely on managing and delivering blog content to Next.js apps.

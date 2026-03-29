# Hello World

Install the `atom-nextjs` package and render your first blog page in under a minute.

## Prerequisites

- A Next.js 14+ project using the App Router
- An Atom project key (grab one from the [Atom dashboard](https://www.atomcms.dev) after creating a project)

## 1. Install the SDK

```bash
npm install atom-nextjs
```

## 2. Add your project key

In your `.env.local` file:

```
ATOM_PROJECT_KEY=atom-your-key-here
```

## 3. Render a blog listing page

Create `app/blog/page.tsx` with the `AtomPage` component. It's an async server component that fetches your posts and renders them automatically.

```tsx
import { AtomPage } from "atom-nextjs";

export default function Blog() {
  return (
    <AtomPage
      baseRoute="/blog"
      projectKey={process.env.ATOM_PROJECT_KEY!}
    />
  );
}
```

`baseRoute` tells Atom where your blog lives so post links point to the right place (for example, `/blog/post-id`).

## 4. Render a single post

Create `app/blog/[id]/page.tsx` to display individual posts using the `Atom` component:

```tsx
import { Atom } from "atom-nextjs";

export default function BlogPost({ params }: { params: { id: string } }) {
  return (
    <Atom
      projectKey={process.env.ATOM_PROJECT_KEY!}
      postId={params.id}
    />
  );
}
```

## Verify it works

Run `npm run dev`, open `http://localhost:3000/blog`, and you should see your posts listed. Click one to view the full article.

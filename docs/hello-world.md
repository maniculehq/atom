# Hello World

Install the `atom-nextjs` package and render your first blog page in under a minute.

## Prerequisites

- A Next.js 14+ project using the App Router
- [Tailwind CSS](https://tailwindcss.com/docs/installation) installed with the [`@tailwindcss/typography`](https://tailwindcss.com/docs/typography-plugin) plugin — Atom components use Tailwind utility classes and `prose` for styling
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

Your project key authenticates requests from the SDK to the Atom API. It's safe in a server-side environment variable because `AtomPage` and `Atom` are both async server components — the key never reaches the browser.

## 3. Render a blog listing page

Create `app/blog/page.tsx` with the `AtomPage` component. It's an async server component that fetches your posts and renders them as a grid of linked cards.

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

Wrapping `AtomPage` in `<Suspense>` lets Next.js stream the shell immediately while posts load. `AtomLoadingSkeleton` provides a built-in loading state, though you can substitute your own component.

### AtomPage props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | — | Your Atom project key (required). |
| `baseRoute` | `string` | — | URL prefix for post links. A post with id `my-first-post` links to `{baseRoute}/my-first-post`. |
| `title` | `boolean` | `true` | Whether to display the project title as an `<h1>` above the post list. |

## 4. Render a single post

Create `app/blog/[id]/page.tsx` to display individual posts using the `Atom` component:

```tsx
import { Atom, AtomArticleSkeleton } from "atom-nextjs";
import { Suspense } from "react";

export default function BlogPost({ params }: { params: { id: string } }) {
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

The `Atom` component fetches a single post by its `postId`, then renders the title, author, date, cover image (if set), and the markdown body compiled to React components via MDX.

### Atom props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `projectKey` | `string` | — | Your Atom project key (required). |
| `postId` | `string` | — | The id of the post to render (required). Comes from the dynamic route segment. |
| `remarkPlugins` | `any[]` | `[]` | Additional [remark](https://github.com/remarkjs/remark) plugins for markdown processing. |
| `rehypePlugins` | `any[]` | `[]` | Additional [rehype](https://github.com/rehypejs/rehype) plugins for HTML post-processing. |

The SDK includes [`remark-gfm`](https://github.com/remarkjs/remark-gfm) (tables, strikethrough, autolinks) and [`rehype-sanitize`](https://github.com/rehypejs/rehype-sanitize) (strips unsafe HTML) by default. Your custom plugins run alongside these.

## 5. Add SEO metadata (optional)

Use `generatePostMetadata` in your single-post page to set the `<title>`, description, author, and keywords from your post data:

```tsx
import { Atom, AtomArticleSkeleton, generatePostMetadata } from "atom-nextjs";
import { Suspense } from "react";
import { Metadata } from "next";

export async function generateMetadata({
  params,
}: {
  params: { id: string };
}): Promise<Metadata> {
  return generatePostMetadata(process.env.ATOM_PROJECT_KEY!, params.id);
}

export default function BlogPost({ params }: { params: { id: string } }) {
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

`generatePostMetadata` returns a Next.js `Metadata` object with `title`, `description` (from the post teaser), `keywords`, and `authors` populated from your post content.

## Verify it works

Run `npm run dev`, open `http://localhost:3000/blog`, and you should see your posts listed. Click one to view the full article.

If the page renders the project title and post cards, everything is wired up correctly. If you see an error message instead, double-check that your `ATOM_PROJECT_KEY` in `.env.local` matches the key shown in the Atom dashboard for your project.

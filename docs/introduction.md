# Introduction

Atom is a headless CMS built specifically for Next.js. The idea is simple: you write and manage blog posts in the Atom dashboard, then drop a couple of React server components into your own Next.js site to render them. No custom API layer to build, no data-fetching logic to wire up yourself.

The system has two parts that work together.

## The Atom Dashboard

The dashboard lives at [cmsatom.netlify.app](https://cmsatom.netlify.app). You sign up, create a **project** (a named collection of posts), and start writing. Each post has a title, author, teaser, optional cover image, keywords, and a Markdown body. The editor is built into the dashboard — no external tools required.

When you create a project, Atom generates a unique **project key** for it. This key is a Bearer token your Next.js site uses to fetch your content. Keep it in an environment variable; it's the only credential the SDK needs.

## The `atom-nextjs` SDK

The SDK is a small npm package you install in your own Next.js application:

```bash
npm install atom-nextjs
```

It exports a set of async React server components and utility functions that talk directly to the Atom API using your project key. Because they run server-side, the key is never exposed to the browser.

Here's what the package exports and what each piece does:

**Components**

- `AtomPage` — renders a list of all posts in your project. Pass it your project key and the base route of your blog (e.g. `/blog`), and it fetches the project data and renders a card grid.
- `Atom` — renders a single post. Pass it your project key and the post ID from the URL, and it fetches and displays the full article with title, cover image, author, date, and formatted Markdown body.
- `AtomBody` — the MDX renderer used internally by `Atom`. You can use it directly if you want to render post content inside your own layout.
- `AtomPostCard` — the individual card component used by `AtomPage`. Import it separately if you want to build a custom listing layout.
- `AtomLoadingSkeleton` / `AtomArticleSkeleton` — loading state placeholders you can use in `Suspense` boundaries while posts load.

**Data-fetching utilities**

- `getProject(projectKey)` — fetches project metadata and a list of post summaries (id, title, teaser, author, dates, image).
- `getPost(projectKey, postId)` — fetches the full content of a single post, including the Markdown body.

**Next.js helpers**

- `generatePostMetadata(projectKey, postId)` — returns a Next.js `Metadata` object populated from the post's title, teaser, keywords, and author. Drop it straight into `generateMetadata` in a page file.
- `generateSitemap(projectKey, blogRoute)` — returns sitemap entries for all posts in your project. Use it inside `sitemap.ts` to keep your sitemap in sync with your content automatically.

## How it fits together

When a visitor loads your blog, `AtomPage` or `Atom` runs on the server, calls the Atom API with your project key, and returns fully rendered HTML. There's no client-side data fetching for the content itself. The API is rate-limited to 30 requests per minute per IP.

The short version: you manage content in one place (the Atom dashboard), and your Next.js site stays focused on presentation. Adding a blog to an existing Next.js project is a matter of two new route files.

# Introduction

Atom is a headless CMS built specifically for Next.js. You write and manage blog posts in the Atom dashboard, then drop a couple of React server components into your own Next.js site to render them. There's no custom API layer to build and no data-fetching logic to wire up yourself.

The system has two parts that work together: the **Atom dashboard**, where content lives, and the **`atom-nextjs` SDK**, which renders that content inside your app.

## Set up your content in the Atom dashboard

The dashboard lives at [cmsatom.netlify.app](https://cmsatom.netlify.app). You sign up, create a **project** (a named collection of posts), and start writing. Each post has a title, author, teaser, optional cover image, keywords, and an MDX body. The editor is built into the dashboard — no external tools required.

When you create a project, Atom generates a unique **project key** for it. This key is a Bearer token your Next.js site uses to fetch your content. Store it in an environment variable; it's the only credential the SDK needs and it never leaves the server.

## Render content with the `atom-nextjs` SDK

The SDK is a small npm package you install in your own Next.js application:

```bash
npm install atom-nextjs
```

It exports async React server components and utility functions that talk directly to the Atom API using your project key. Because they run server-side, the key is never exposed to the browser.

### Components

These are the building blocks for your blog routes. For most projects you'll only need these two:

- `AtomPage` — renders a list of all posts in your project. Pass it your project key and the base route of your blog (e.g. `/blog`), and it fetches the project data and renders a card grid.
- `Atom` — renders a single post. Pass it your project key and the post ID from the URL, and it fetches and displays the full article with title, cover image, author, date, and rendered MDX body.

The following components are available for custom layouts and loading states:

- `AtomBody` — the MDX renderer used internally by `Atom`. Use it directly only if you want to render post content inside a fully custom layout.
- `AtomPostCard` — the individual card component used by `AtomPage`. Import it separately only if you need a custom listing layout.
- `AtomLoadingSkeleton` / `AtomArticleSkeleton` — loading-state placeholders for use in `Suspense` boundaries while posts load.

### Data-fetching utilities

These functions underpin the components above. Call them directly when you need raw data rather than pre-rendered markup.

- `getProject(projectKey)` — fetches project metadata and a list of post summaries (id, title, teaser, author, dates, image).
- `getPost(projectKey, postId)` — fetches the full content of a single post, including the raw MDX body.

### Next.js helpers

These helpers keep your Next.js-specific metadata and sitemap in sync with your content automatically.

- `generatePostMetadata(apiKey, postId)` — returns a Next.js `Metadata` object populated from the post's title, teaser, keywords, and author. Drop it straight into `generateMetadata` in a page file.
- `generateSitemap(projectKey, blogRoute)` — returns sitemap entries for all posts in your project. Use it inside `sitemap.ts` to keep your sitemap current without any manual updates.

## How it all fits together

When a visitor loads your blog, `AtomPage` or `Atom` runs on the server, calls the Atom API with your project key, and returns fully rendered HTML. There's no client-side data fetching for the content itself.

The result is a clean separation of concerns: you manage content in one place (the Atom dashboard), and your Next.js site stays focused on presentation. Adding a blog to an existing Next.js project takes two new route files.

One operational detail worth knowing upfront: the Atom API is rate-limited to **30 requests per minute per IP** using a sliding window. When the limit is exceeded, the API returns HTTP 200 with `success: false` and `message: "Too many requests."` — not an HTTP 429. For most blogs this limit is invisible, but it is worth keeping in mind if you are pre-rendering many posts at build time from a single IP.

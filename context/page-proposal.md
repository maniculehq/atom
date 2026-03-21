# Page Proposal

Proposed documentation pages for the Atom CMS project.

## Pages

- **getting-started** — Introduction to Atom CMS, the platform + SDK architecture, sign-up flow, and obtaining a project key.
- **sdk-setup** — Installing `atom-nextjs`, configuring TailwindCSS, and setting the `ATOM_PROJECT_KEY` environment variable.
- **atom-page-component** — The `AtomPage` server component: props (`projectKey`, `baseRoute`, `title`), usage example, and `AtomLoadingSkeleton`.
- **atom-article-component** — The `Atom` server component: props (`projectKey`, `postId`, `remarkPlugins`, `rehypePlugins`), MDX rendering, `generatePostMetadata`, and `AtomArticleSkeleton`.
- **sitemap-generation** — Using `generateSitemap()` to produce Next.js sitemap entries from project posts.
- **api-reference** — Full REST API reference: all auth, project, and post endpoints with request/response shapes, authentication methods, and rate limiting.
- **managing-projects** — Creating, listing, and deleting projects in the dashboard; understanding the project key; plan limits.
- **managing-posts** — Creating, editing, and deleting posts; the Markdown/MDX editor; field reference (title, author, body, teaser, keywords, image); plan body-length limits.
- **authentication** — Sign-up/sign-in flows, Lucia v3 session management, password requirements, profile updates, and account deletion.
- **pricing-plans** — The three plan tiers (Single, Startup, Business), their limits, and current availability.
- **configuration** — Reference for all environment variables: `HASH_SALT`, `MONGO_DB_URI`, `ATOM_PROJECT_KEY`, `UPSTASH_REDIS_REST_URL`, `UPSTASH_REDIS_REST_TOKEN`, `NEXT_PUBLIC_ENV`.
- **local-development** — Running the platform locally, linking the SDK package with `npm link`, and using the Bruno API collection for testing.

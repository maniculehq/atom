# Answers

## Q: What is the production deployment target and URL?

The codebase shows inconsistent production URLs. The API base URL in `lib/contants.tsx` points to `https://cmsatom.netlify.app/api` (Netlify), while SEO files (`app/sitemap.ts`, `app/robots.ts`) reference `https://atomcms.vercel.app` (Vercel). The SDK constants file has a comment suggesting a planned migration to `https://www.atomcms.dev/api`. The deployment situation appears to be in flux or transitioning between hosts.

## Q: Is the `atom-nextjs` package published to npm separately or only used via npm link?

The `atom-nextjs` package (v0.3.1) is published on npm and used as a regular dependency by the main app (`"atom-nextjs": "^0.3.1"` in package.json). It is also developed within this monorepo under `packages/atom-nextjs/` and can be linked locally during development using `npm link`.

## Q: Is the billing/payments system implemented?

The billing system is not yet implemented. The plan structure exists in the data model with three tiers (single, startup, business), and plan limits are enforced in API routes. However, only the free "single" plan is active — the paid plans have `disabled: true`, the billing page shows "Coming soon", and there is no payment provider integration.

## Q: What is the `contants.tsx` file's intentional filename?

The filename `lib/contants.tsx` is a typo (should be "constants"). The misspelling has propagated throughout the codebase via imports. Similarly, the type `UserDocumnetProjectsCreator` in `lib/types.ts` is a typo for "UserDocumentProjectsCreator".


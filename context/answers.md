# Answers

## Q: Where is the app deployed — Netlify or Vercel?

Both platforms appear to be in use or in transition. The API base URL in both the main app (`lib/contants.tsx`) and the SDK (`packages/atom-nextjs/src/lib/constants.ts`) points to `cmsatom.netlify.app`, while SEO files (`app/robots.ts`, `app/sitemap.ts`) reference `atomcms.vercel.app`. A comment in the SDK constants suggests a planned migration to `atomcms.dev`. Documentation should treat `cmsatom.netlify.app` as the current API host.

## Q: Is billing/payments implemented?

Billing is partially implemented. The plan system exists in the data model and plan limits (max projects, max posts, max body length) are actively enforced in API routes. However, there is no payment processing integration — no Stripe, no webhooks, no checkout flow. The billing page shows "Coming soon..." and paid plans (`startup`, `business`) are marked `disabled: true`. All users default to the free "single" plan.

## Q: What is the relationship between the main app's blog and the SDK?

The Atom website dogfoods its own SDK. The `/blog` routes (`app/blog/page.tsx`, `app/blog/[id]/page.tsx`) import and use `AtomPage`, `Atom`, and `generatePostMetadata` from the `atom-nextjs` package, authenticating with `process.env.ATOM_PROJECT_KEY`. The sitemap generation also uses the SDK's `generateSitemap` function.


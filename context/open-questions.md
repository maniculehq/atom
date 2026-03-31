# Open Questions

## Q1: Where is the app deployed — Netlify or Vercel?

The codebase references two different hostnames in different locations, creating ambiguity about the canonical deployment.

### Options
1. **Netlify is the primary deployment** — The `baseAPIRoute` constant in `lib/contants.tsx:L28-L30` uses `https://cmsatom.netlify.app/api` for production. The SDK's hardcoded constant in `packages/atom-nextjs/src/lib/constants.ts` also points to `https://cmsatom.netlify.app/api`. This is the URL that actual API requests go to.
2. **Vercel is the primary deployment** — The `app/robots.ts` references `https://atomcms.vercel.app` as the host and sitemap URL. The `app/sitemap.ts` uses `https://atomcms.vercel.app/blog` as the base. These are SEO-facing URLs.
3. **Both are active / in transition** — The app may be deployed to both platforms simultaneously, or may have been migrated from one to the other without fully updating all references. The SDK constant has a comment `// CHANGE THIS TO https://www.atomcms.dev/api` suggesting a planned domain migration.

### Recommendation
**Option 3: Both are active / in transition** — The evidence shows Netlify is used for the API (functional references in both the main app and SDK), while Vercel URLs appear in SEO files. The comment in `packages/atom-nextjs/src/lib/constants.ts` about changing to `atomcms.dev` suggests a domain migration is planned. Documentation should note `cmsatom.netlify.app` as the current API base but acknowledge the Vercel references.

---

## Q2: Is billing/payments implemented?

### Options
1. **Billing is not implemented** — The billing page at `app/app/settings/billing/page.tsx` displays "Coming soon..." with no payment integration code. In `lib/contants.tsx:L61-L86`, both `startup` and `business` plans have `disabled: true` and `active: false`. There is no Stripe or payment provider integration anywhere in the codebase.
2. **Billing is partially implemented** — The plan system exists in the data model (`UserDocument.plan` field, `planDetails` with pricing), and plan limits are enforced in API routes (e.g., `app/api/posts/create/route.ts` checks `userPlan.max_body_length` and `userPlan.max_projects`). The infrastructure is in place but payment processing is not connected.
3. **Billing is fully implemented but hidden** — Unlikely; no payment provider SDK or webhook handlers exist in the codebase.

### Recommendation
**Option 2: Billing is partially implemented** — The plan enforcement logic is real and functional (enforced in post creation, project creation, and post update routes), but there is no way for users to upgrade plans since no payment integration exists. All users default to the "single" (free) plan. Documentation should describe the plan system as it exists while noting that paid plan upgrades are not yet available.

---

## Q3: What is the relationship between the main app's blog and the SDK?

### Options
1. **The main app dogfoods its own SDK** — The Atom website itself uses `atom-nextjs` to render its own blog at `/blog`. This is evidenced by `app/blog/page.tsx` importing `AtomPage` from `atom-nextjs` and `app/sitemap.ts` using `generateSitemap` from `atom-nextjs`, both using `process.env.ATOM_PROJECT_KEY`.
2. **The blog routes are demo/example code** — The blog pages might exist primarily as a demonstration of the SDK for potential users.
3. **The blog is independent of the SDK** — Incorrect; the imports clearly show SDK usage.

### Recommendation
**Option 1: The main app dogfoods its own SDK** — The Atom website eats its own dogfood by using the `atom-nextjs` SDK to power its own blog section. This is a strong pattern to highlight in documentation, as it demonstrates real-world SDK usage directly in the source repo.


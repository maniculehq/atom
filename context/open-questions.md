# Open Questions

## Q1: What is the production deployment target and URL?

### Options
1. **Netlify at cmsatom.netlify.app** — The `baseAPIRoute` in `lib/contants.tsx:L32` uses `https://cmsatom.netlify.app/api` for production. The SDK's `packages/atom-nextjs/src/lib/constants.ts` also hardcodes this URL with a comment mentioning `https://www.atomcms.dev/api` as a future target.
2. **Vercel at atomcms.vercel.app** — The `app/sitemap.ts` and `app/robots.ts` files reference `https://atomcms.vercel.app` as the canonical host. The landing page metadata says "Atom - The NextJS CMS".
3. **Both/migration in progress** — There may be a migration from Netlify to Vercel, or the sitemap/robots references are outdated.

### Recommendation
**Option 3: Both/migration in progress** — The codebase shows inconsistency: the API base URL points to Netlify (`cmsatom.netlify.app`) while SEO files point to Vercel (`atomcms.vercel.app`). The SDK constants file has a comment "CHANGE THIS TO https://www.atomcms.dev/api" suggesting a planned domain change. This suggests the deployment situation is in flux. Documentation should note both URLs and clarify with the developer.

---

## Q2: Is the `atom-nextjs` package published to npm separately or only used via npm link?

### Options
1. **Published to npm** — The `package.json` lists `"atom-nextjs": "^0.3.1"` as a dependency, and it has a `prepare` script that builds via `tsdx build`. The README mentions `npm i atom-nextjs@latest`.
2. **Local only via npm link** — The README has instructions for linking locally with `npm link`.
3. **Both** — Published to npm for end users, but linked locally during development.

### Recommendation
**Option 3: Both** — The package is published on npm (version 0.3.1, referenced in `lib/contants.tsx:L85` as `npm i atom-nextjs@latest`). It's also developed within this monorepo and can be linked locally for development. The main app uses the published version as a dependency.

---

## Q3: Is the billing/payments system implemented?

### Options
1. **Not implemented** — The billing page at `app/app/settings/billing/page.tsx` simply renders "Coming soon...". The `startup` and `business` plans have `disabled: true` in `lib/contants.tsx`.
2. **Partially implemented** — Plan types exist in the data model and are enforced in API routes (post creation checks plan limits).
3. **Planned for future** — The plan structure is in place but no payment integration exists.

### Recommendation
**Option 3: Planned for future** — The plan system is structurally complete (types, limits, enforcement in API routes) but only the free "single" plan is active. Paid plans are disabled (`disabled: true`), and the billing page shows "Coming soon". There's no payment provider integration (no Stripe, etc.).

---

## Q4: What is the `contants.tsx` file's intentional filename?

### Options
1. **Typo — should be `constants.tsx`** — The file is at `lib/contants.tsx` which appears to be a misspelling of "constants". Similarly, `UserDocumnetProjectsCreator` in `lib/types.ts` has "Documnet" instead of "Document".
2. **Intentional naming** — Unlikely but possible.
3. **Legacy naming that hasn't been refactored** — The typo exists throughout the codebase (all imports reference `contants`), so renaming would require updating many files.

### Recommendation
**Option 1: Typo** — This is clearly a typo that has propagated throughout the codebase. Documentation should use the correct spelling "constants" when referring to concepts, but reference the actual filename `contants.tsx` when pointing to file paths.


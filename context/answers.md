# Answers

## Q: What is the canonical deployment URL for the Atom dashboard?

The primary deployment URL is `cmsatom.netlify.app`. This is hardcoded in the `atom-nextjs` SDK's `baseAPIRoute` constant (`packages/atom-nextjs/src/lib/constants.ts`) and in the main app's production API base (`lib/contants.tsx:L28-L30`). The `atomcms.vercel.app` URL appears in `app/robots.ts` and `app/sitemap.ts` and may be a secondary or stale deployment.

## Q: What is the typo in `lib/contants.tsx` — intentional or accidental?

It is an accidental typo. The file is named `contants.tsx` instead of `constants.tsx`. All imports throughout the codebase reference `@/lib/contants`. Documentation should use the actual filename `lib/contants.tsx` when pointing to paths but may note the misspelling.

## Q: Is the `Atom` component export being renamed to `AtomPost`?

Yes. The most recent commit (`5e79ae6`) explicitly renames the component from `Atom` to `AtomPost` for clarity. The file `packages/atom-nextjs/src/components/Atom.tsx` now exports `AtomPost`. The SDK's `index.tsx` re-exports it as `Atom` for backward compatibility, but `AtomPost` is the canonical name going forward. The app's blog page (`app/blog/[id]/page.tsx`) already uses `AtomPost`.

## Q: Are the `startup` and `business` plans intended to be active?

They are future plans, currently disabled. Both have `disabled: true` in `lib/contants.tsx`. The billing page (`app/app/settings/billing/page.tsx`) shows "Coming soon...". Plan-based enforcement logic (max projects, max posts, max body length) is already implemented in the API routes. Only the free `single` plan is currently active.

## Q: What testing framework and coverage exists?

Testing is not yet implemented. No test files exist in either the main app or the `atom-nextjs` package. The SDK has a test script (`tsdx test --passWithNoTests`) but no actual test files. The main app's `package.json` has no test script.

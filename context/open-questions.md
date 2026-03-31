# Open Questions

## Q1: What is the canonical deployment URL for the Atom dashboard?

### Options
1. **`cmsatom.netlify.app`** — This is hardcoded in `packages/atom-nextjs/src/lib/constants.ts` as the `baseAPIRoute` and is what the SDK uses to fetch data. Also referenced in `lib/contants.tsx:L29` as the production API base.
2. **`atomcms.vercel.app`** — This is used in `app/robots.ts` (host + sitemap URL) and `app/sitemap.ts` as the site URL for sitemap generation.
3. **Both are valid** — The project may be deployed to both platforms, with Netlify as the primary API host and Vercel as the primary site host.

### Recommendation
**Option 1: `cmsatom.netlify.app`** — The SDK hardcodes this URL for all API calls, making it the primary deployment. The Vercel URL in robots/sitemap may be stale or secondary. The `baseAPIRoute` in `lib/contants.tsx` also uses this for production.

---

## Q2: What is the typo in `lib/contants.tsx` — intentional or accidental?

### Options
1. **Accidental typo** — The file is named `contants.tsx` instead of `constants.tsx`. All imports reference `@/lib/contants` throughout the codebase (grep confirms ~15+ imports).
2. **Intentional shortening** — An unlikely but possible deliberate naming choice.
3. **Legacy name** — Started as a typo and is now too deeply used to rename without a refactor.

### Recommendation
**Option 1: Accidental typo** — It's clearly meant to be "constants". The file contains plan definitions, nav options, and configuration values — classic constants. All documentation should refer to it as `lib/contants.tsx` (matching the actual filename) but may note the typo.

---

## Q3: Is the `Atom` component export being renamed to `AtomPost`?

### Options
1. **Rename in progress** — The most recent git commit (`5e79ae6`) says "refactor: rename Atom component to AtomPost for clarity". The component file `packages/atom-nextjs/src/components/Atom.tsx` exports `AtomPost`. The `index.tsx` still exports `Atom` (the old name alongside the component imported from `Atom.tsx`).
2. **Both names are supported** — `Atom` as a legacy alias and `AtomPost` as the new name.
3. **Only `AtomPost` should be used** — The blog page at `app/blog/[id]/page.tsx` already imports `AtomPost`.

### Recommendation
**Option 3: Only `AtomPost` should be used** — The recent commit explicitly renames for clarity. The SDK's `index.tsx` exports `Atom` (which is actually the `AtomPost` component from `Atom.tsx`), so both names work, but `AtomPost` is the canonical name going forward. Documentation should use `AtomPost`.

---

## Q4: Are the `startup` and `business` plans intended to be active?

### Options
1. **Future plans, currently disabled** — In `lib/contants.tsx`, both `startup` ($3.99) and `business` ($11.99) have `disabled: true` and `active: false`. The billing page shows "Coming soon...".
2. **Permanently disabled** — The plans exist only for display on the pricing page.
3. **Planned for near-term activation** — The infrastructure supports plan-based limits (max_projects, max_docs, max_body_length) and the code checks these limits in post creation/update routes.

### Recommendation
**Option 1: Future plans, currently disabled** — The pricing page renders all three plans, the billing page says "Coming soon...", and plan limit enforcement is already coded into the API. These are clearly intended to be activated when billing integration is added. Documentation should mention the free tier is currently the only active plan.

---

## Q5: What testing framework and coverage exists?

### Options
1. **No tests in the main app** — There are no test files visible in the main Next.js app directory structure. No test scripts in root `package.json`.
2. **Tests only in atom-nextjs package** — `packages/atom-nextjs/package.json` has `"test": "tsdx test --passWithNoTests"`, which uses Jest under the hood, but `--passWithNoTests` suggests no tests exist yet.
3. **Testing is not yet implemented** — Neither the main app nor the SDK package has actual test files.

### Recommendation
**Option 3: Testing is not yet implemented** — No test files found anywhere in the repository. The SDK has a test script but uses `--passWithNoTests`. The main app has no test script. Documentation should note this.

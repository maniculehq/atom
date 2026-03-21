# Open Questions

## Architecture & Design

1. **`validateProjectKey.ts` appears unused**: `lib/server/utils/validateProjectKey.ts` validates a `project_key` against `UserDocuments`, but `UserDocument` doesn't store `project_key` — only `Project` does. This function seems to be dead code or a bug. Which collection was it intended to query?

2. **`/api/projects/get/single` dual-auth ambiguity**: This route accepts both session cookie auth AND Bearer token. The client SDK uses `/api/projects/get/single/client` (Bearer only), while the dashboard uses the session-cookie variant. Should these be better separated or documented?

3. **`validateRequestFetchUser.ts` usage**: `lib/server/utils/validateRequestFetchUser.ts` is defined but not currently called anywhere in the visible routes. Is it a helper that's planned for future use or was it replaced?

4. **billing page is a placeholder**: `app/app/settings/billing/page.tsx` only says "Coming soon...". Are there plans for Stripe integration or other billing infrastructure?

5. **Startup and Business plans are `disabled: true`**: Paid plans are marked disabled. Is billing ever planned to be enabled? This affects the pricing page UI (shows "Coming soon" buttons).

## SDK & API

6. **SDK `baseAPIRoute` is hardcoded**: `packages/atom-nextjs/src/lib/constants.ts` hardcodes `https://cmsatom.netlify.app/api`. If the platform domain changes, all SDK consumers need to upgrade. Is this intentional (simpler) or should it be configurable?

7. **Post body length enforcement in SDK**: The SDK doesn't validate body length client-side. Enforcement is server-side only. Is there a plan to surface cleaner error messages to SDK consumers?

8. **Max posts (`max_docs`) not enforced**: The `max_docs` field in `planDetails` exists but the post creation route (`app/api/posts/create/route.ts`) only checks `max_body_length`, not total post count. Is total post count enforcement intentional to skip, or a bug?

## Security

9. **HASH_SALT (pepper) rotation**: There's no mechanism to rotate the `HASH_SALT`. If it needs to change, all passwords become invalid. Is this a known limitation?

10. **Rate limiting on non-API routes**: `middleware.ts` only applies rate limiting to `/api/*`. The dashboard (`/app/*`) and auth pages have no rate limiting. Is this intentional?

11. **Session cookie expiry**: Lucia is configured with `expires: false` (`lib/server/lucia/init.ts:L12`), meaning sessions never expire. Is this the intended behavior?

## Development

12. **No test coverage**: There are zero test files for the main application. Only the SDK has a test script (`tsdx test --passWithNoTests`) which also passes with no tests. Is testing planned?

13. **`NEXT_PUBLIC_ENV` vs `NODE_ENV`**: The app uses `NEXT_PUBLIC_ENV` (`"prod"` or `"dev"`) to switch `baseAPIRoute`, but uses the standard `NODE_ENV` for the session cookie `secure` flag. This could cause subtle bugs if one is set incorrectly.

14. **Missing `creator_uid` on post**: `Post` type has `creator_uid` field, but in the create post route, this field is not populated in the `$push` operation. It always reflects `project.creator_uid` (owner). Is the `Post.creator_uid` field ever expected to differ from project owner?

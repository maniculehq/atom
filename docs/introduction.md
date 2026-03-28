# Introduction

Atom is a headless CMS built for Next.js. You write and manage blog posts in the Atom dashboard, then render them in your Next.js app using the `atom-nextjs` SDK.

## How it works

1. **Create a project** in the Atom dashboard. Each project holds a collection of blog posts and has a unique project key.
2. **Install the SDK** (`atom-nextjs`) in your Next.js app.
3. **Drop in two components** (one for your blog list page, one for individual posts) and pass them your project key. Atom handles fetching, rendering Markdown, and generating metadata.

That's the whole integration. Your content lives in Atom, your site stays in your repo, and the SDK connects the two at render time using server components.

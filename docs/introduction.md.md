# Introduction

Atom is a headless CMS built for Next.js. It has two parts: a dashboard where you write and manage markdown blog posts, and an npm package (`atom-nextjs`) that renders them in your Next.js app.

You create a **project** in the dashboard, write your posts in the markdown editor, and grab the project key. Then install `atom-nextjs`, drop in the `AtomPage` and `Atom` server components, pass them your key, and your blog is live. Atom handles the content. Your app keeps full control of layout, styling, and routing.

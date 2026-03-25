---
source: https://cmsatom.netlify.app
fetched: 2026-03-25T06:11:01.992Z
title: Atom - The NextJS CMS
---
# Ship blogs and articles in minutes.

Create, edit, and publish a fully functioning blog in NextJS quickly using Atom.

[Get started](/signin)

npm i atom-nextjs@latest

# Get started with just two files

### Create your blog page route and add the blog page component.

Create your blog file (/app/blog/page.tsx), import your API key, and our pre-made blog component. This will handle the blog page, where users can select what blog post to read.

```javascript
// app/blog/page.tsx

import { AtomPage } from 'atom-nextjs';
import { MyAppContainer } from '@/...';

export const metadata = {
  title: 'Blog',
};

export default function Blog() {
  return (
    <MyAppContainer>
      <AtomPage baseRoute="/blog" projectKey={process.env.ATOM_PROJECT_KEY!} />
    </MyAppContainer>
  );
}

```

### Create your post page route, and add the post component.

Create your post file (/app/blog/\[id\]/page.tsx), import your API key, and our pre-made post component. This component will handle the page that includes your blog/post text.

```javascript
// app/blog/[id]/page.tsx

import { Atom, generatePostMetadata } from 'atom-nextjs';
import { MyAppContainer } from '@/...';

export type BlogParams = { params: { id: string } };

export const generateMetadata = async ({ params }: BlogParams) => {
  const metadata = await generatePostMetadata(
    process.env.ATOM_PROJECT_KEY!,
    params.id
  );

  return metadata;
};

export default async function BlogPage({ params }: BlogParams) {
  return (
    <MyAppContainer>
      <Atom projectKey={process.env.ATOM_PROJECT_KEY!} postId={params.id} />
    </MyAppContainer>
  );
}

```

Ready to get started?

# Sign up for free

[Get started](/signup)

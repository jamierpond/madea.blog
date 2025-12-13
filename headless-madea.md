# Headless Madea: Bring Your Own Frontend

A few weeks ago I wrote about [using GitHub as a CMS](/github-as-cms.md). Push markdown, it's live. No database, no build step, no configuration.

But here's the thing: `madea.blog` is just *one* way to render that content. What if you want a 90s Geocities aesthetic? A terminal-based UI? A 3D WebGL blog where posts float in space?

Enter `madea-blog-core`.

## The Split

I realized early that the "fetch markdown from GitHub" logic and the "render it pretty" logic don't need to live together. So I split them:

1. **`madea-blog-core`**: The brain. Handles GitHub API, file tree traversal, caching, rate limits. Doesn't know what HTML is.
2. **Your App**: The body. Takes data from the core, renders it however you want.

This is "headless" in the truest sense. Except the CMS is free (it's GitHub), the data is yours (it's a git repo), and the library is a single npm install.

## Quick Start

```bash
npm install madea-blog-core -- coming soon
```

Then pick your pattern.

### The Controller Pattern (Easy Mode)

This handles routing, 404s, and data fetching. You just supply the views.

```tsx
// app/[...slug]/page.tsx
import { renderMadeaBlogPage } from 'madea-blog-core';
import { GitHubDataProvider } from 'madea-blog-core/providers/github';

const provider = new GitHubDataProvider({
  username: 'yourname',
  repo: 'madea.blog',
  token: process.env.GITHUB_TOKEN
});

const MyArticleView = ({ article }) => (
  <div className="prose">
    <h1>{article.title}</h1>
    <div dangerouslySetInnerHTML={{ __html: article.content }} />
  </div>
);

export default async function Page({ params }) {
  return renderMadeaBlogPage({
    dataProvider: provider,
    articleView: MyArticleView,
  }, params);
}
```

That's a complete blog. The library handles everything else.

### The Data Provider Pattern (God Mode)

Want total control? Just use the provider directly.

```typescript
import { GitHubDataProvider } from 'madea-blog-core/providers/github';

const api = new GitHubDataProvider({
  username: 'jamierpond',
  repo: 'madea.blog'
});

const posts = await api.getArticleList();
const post = await api.getArticle('hello-world.md');

console.log(post.commitInfo.authorName); // "Jamie Pond"
console.log(post.commitInfo.date);       // "2024-01-15T..."
```

Build your own routing. Your own caching. Your own everything. The provider just fetches and normalizes the data.

## Case Study: The Code Golf Blog

I challenged myself: how minimal can a functional blog get?

The result was `[golf.blog.pond.audio]("https://golf.blog.pond.audio")`. Same `madea-blog-core` package, stripped to the bones.

## Why Go Headless?

**Custom Branding**: You're not stuck with my styles. Build whatever aesthetic you want.

**Custom Routing**: Want your blog at `/writing` instead of `/`? Want posts at `/thoughts/[year]/[slug]`? Your app, your routes.

**Hybrid Data**: Mix GitHub content with Spotify listening history, Strava runs, or any other API. The provider is just one data source.

**Static Builds**: Use `getArticleList` inside `generateStaticParams` to statically generate everything at deploy time. Zero runtime GitHub API calls.

## Local Development

The core also includes `LocalFsDataProvider`. Point it at a folder instead of GitHub.

```typescript
import { LocalFsDataProvider } from 'madea-blog-core/providers/local-fs';

const provider = new LocalFsDataProvider({
  contentDir: './content'
});
```

Perfect for:
- Developing your theme offline
- Previewing posts before pushing
- Internal tools that don't need the internet
- Storing your markdown with your blog.

Switching from local to GitHub is one line.

## The Meta Continues

This post is served via GitHub's API. But it could just as easily be served by *your* app, using the same underlying content, with completely different styling.

That's the point. The content lives in Git. How you render it is up to you.

Most "Headless CMS" platforms are expensive SaaS products that hold your data hostage behind an API key. This is different. The CMS is GitHub (free). The data is yours (it's a git repo). The library is open source.

Headless, but actually free.

---

*The original post explained how to abuse GitHub as a CMS. This post explains how to abuse that abuse. The infrastructure keeps getting more cursed, but somehow it keeps working.*

*`git push` remains your deploy button. Now you just have more options for what happens after.*

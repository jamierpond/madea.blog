# Your new blog: Abusing GitHub as a CMS

I spent a few hours over a few weekends building a blog platform that has no database, no markdown processor on the client, no build step, and no configuration. You just push markdown to GitHub and it's live.

Yeah, I'm using GitHub as a CMS. And it's kind of perfect.

## The Problem with Blogging Platforms

Every few years I get the urge to start a blog. The cycle is always the same:

1. Spend 3 hours comparing Static Site Generators
2. Choose one (currently: Next.js, Astro, or whatever is trending)
3. Spend 2 days configuring it
4. Write one post
5. Never touch it again because deployment is annoying

The problem isn't writing. The problem is everything *around* writing. I just want to write markdown and have it appear on the internet. Why is this so hard?

## The Stupid Simple Solution

What if your blog was just... a GitHub repository?

```bash
# This is literally all you do
gh repo create madea.blog --public --clone
cd madea.blog
echo "# My First Post" > hello-world.md
git add .
git commit -m "first post"
git push
```

That's it. Your blog is live at `yourusername.madea.blog`.

No build step. No deployment. No configuration files. No database. Just push markdown to `yourusername/madea.blog` and it's published.

## How It Works

The entire "CMS" is the GitHub API. Here's the architecture:

**Subdomain Routing**
```typescript
// middleware.ts
const hostname = request.headers.get('host') || '';
const username = hostname.split('.')[0];
request.headers.set('x-github-username', username);
```

Each subdomain maps to a GitHub user. `alice.madea.blog` → `alice/madea.blog` repo.

**Content Fetching**
```typescript
const repo = `${username}/madea.blog`;
const files = await fetch(
  `https://api.github.com/repos/${repo}/git/trees/${branch}?recursive=1`
);
```

No database. No markdown files in my repository. Just API calls to GitHub.

**Markdown Rendering**
```typescript
const content = await fetch(
  `https://api.github.com/repos/${repo}/contents/${file}`
);
const decoded = Buffer.from(content, 'base64').toString('utf-8');
```

GitHub stores your markdown. GitHub versions it. GitHub backs it up. I just fetch and render it.

## Why This Actually Works

**GitHub is a better CMS than most CMS platforms:**

- **Version Control**: Built in. Every edit is tracked. Want to revert a post? `git revert`.
- **API**: Fast, reliable, globally distributed. Rate limits are generous (5000 req/hour authenticated).
- **Storage**: Unlimited for text files. Markdown is tiny.
- **Auth**: GitHub's auth. Want to add editors? Add collaborators.
- **Webhooks**: Want to trigger builds? GitHub webhooks.
- **Mobile Editing**: GitHub mobile app. Edit posts from your phone.

**It's stupidly cheap to run:**
- No database → No database hosting costs
- Static content → Edge cache everything
- API calls → GitHub's infrastructure (free)
- Hosting → Vercel free tier handles thousands of users

## The Meta Part

This article you're reading? It's a markdown file in my `madea.blog` repository.

I wrote it in vim, committed it, and pushed. It was live in 2 seconds.

The article about using GitHub as a CMS is *served from GitHub as a CMS*.

## Edge Cases & Limitations

**Rate Limits**: GitHub API allows 5000 requests/hour (authenticated). With edge caching, you'd need serious traffic to hit this. And if you have that kind of traffic, you can afford GitHub Enterprise.

**Build Times**: There is no build. It's instant. Push and it's live (after cache revalidation - 10 minutes by default).

**Rich Media**: GitHub repos can store images. Reference them in your markdown. Or use GitHub as markdown storage and S3/CDN for images.

**Private Posts**: Make the repo private. The blog won't work (GitHub API won't return private repo contents without auth). Want a private blog? Add auth to the Next.js app and use a GitHub token.

## Is This Cursed?

Maybe. But also maybe not?

This is way less cursed than S3 as a database using JSON files.

Is using GitHub as a CMS really that cursed? It's literally designed to store and version text files and provide an API to access them.

## Why I Actually Like This

I've tried many blogging platforms, but they all have the same problem: friction between writing and publishing.

This has zero friction. I write markdown in my editor of choice. I commit. I push. It's live.

No CMS login. No web editor. No build step waiting. No deployment pipeline. No configuration.

Just `git push`.

## Try It Yourself

1. Create a repo called `madea.blog` in your GitHub account
2. Add a markdown file: `echo "# Hello World" > test.md`
3. Push it
4. Visit `yourusername.madea.blog`

That's it. You have a blog.

Want to add another post? Push another markdown file. Want to edit a post? Edit the markdown and push. Want to delete a post? Delete the file and push.

It's just Git.

## The Code

The entire platform is a few hundred lines of Typescipt.
- Next.js API routes that fetch from GitHub
- Middleware for subdomain routing
- React components that render markdown
- That's it

[Check it out on GitHub](https://github.com/jamierpond/madea.blog) (meta inception continues...)

## Is This The Future?

Probably not. But it works, it's fast, it's free, and most importantly: **it gets out of the way**.

I'm not building a blogging platform anymore. I'm just writing markdown and pushing to Git.

Like it should be.

---

*This post was written in vim, and is currently being served to you via an unholy combination of GitHub's API, Vercel's edge network, and the sheer audacity of using version control as a content management system.*

*Welcome to madea.blog - where Git is your CMS and `push` is your deploy button.*

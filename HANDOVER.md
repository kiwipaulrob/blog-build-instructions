# Blog Build — Agent Handover Document

**Prepared:** June 2026  
**Last updated:** June 2026  
**Prepared by:** Tasklet AI agent  
**GitHub account:** kiwipaulrob  
**For:** Any agent continuing build, maintenance, or feature work on these blogs

---

## ⚠️ Strategy Update — June 2026

**Cloudflare Pages has been retired. GitHub Pages is now the standard for all three blogs.**

A sister project (heargodandevil.com — a generated static music site) hit Cloudflare Pages' **25 MB artifact limit** in production — Git history caused pack files to exceed the limit, breaking deployments entirely. While a text blog grows more slowly, the constraint is architectural and applies to any Git-backed project with meaningful history.

Combined with the dual-auth complexity of the Cloudflare build (Cloudflare Access front door + GitHub OAuth PKCE — two separate auth layers), and the fact that two of three blogs were already on GitHub Pages, the decision was made to standardise on GitHub Pages for all builds.

**What this means for an incoming agent:**

| Blog | Was | Now |
|---|---|---|
| Donald from Dunedin | GitHub Pages | GitHub Pages (no change) |
| My Audio Projects | GitHub Pages | GitHub Pages (no change) |
| My Audio Projects CF | Cloudflare Pages (**active focus**) | **RETIRED** — use `myaudioprojects/` instead |

The `myaudioprojects-cf/` folder is kept locally for reference. Do not deploy it. The `myaudioprojects/` GitHub Pages build is the canonical audio blog going forward.

**Deployment checklist for `myaudioprojects/` when ready:**
1. Create GitHub repo `kiwipaulrob/myaudioprojects`, push `myaudioprojects/` to it
2. Enable GitHub Pages (main branch, root folder)
3. Add CNAME file with `myaudioprojects.com`, set DNS A records (185.199.108–111.153)
4. Set up Netlify: connect repo → Identity → invite-only → GitHub provider → Git Gateway → invite blog owner's email
5. Update `admin/config.yml`: replace `GITHUB_USERNAME` placeholder with actual username
6. Transfer repo to blog owner when ready; update `admin/config.yml` again with their username

---

## Purpose of This Document

Paul Robertson is building simple personal blogs for himself and friends. He uses a Tasklet AI agent to do the code and configuration work. This document captures everything an incoming agent needs to take over — the full picture of what has been built, how everything works, where the files live, what decisions were made, what was ruled out and why, and how Paul expects to be worked with.

Read this document in full before doing anything. Do not make assumptions from partial knowledge.

---

## What These Blogs Are

Three blog builds. Each is a simple personal blog — Paul's mental model is "a cheap dumbed down WordPress." The day-to-day use case for the blog owner: visit `/admin`, log in, write a post, hit Publish. No exposure to GitHub, code, or terminals.

Paul builds these on his own GitHub account, then transfers the repo to the blog owner once done.

| Blog | Domain | Hosting | Auth method | Status |
|---|---|---|---|---|
| Donald from Dunedin | donaldfromdunedin.com | GitHub Pages | Netlify Identity | Active |
| My Audio Projects | myaudioprojects.com | GitHub Pages | Netlify Identity | Active — canonical audio blog |
| My Audio Projects (CF variant) | myaudioprojects.com | Cloudflare Pages | Cloudflare Access + GitHub OAuth | **Retired June 2026** |

All three builds are local template builds — none have been deployed to a live domain yet. The `myaudioprojects/` GitHub Pages build is the current focus. The `myaudioprojects-cf/` Cloudflare Pages build is retired and should not be deployed.

---

## Stack

| Layer | Choice | Notes |
|---|---|---|
| Static site generator | Jekyll 4.3 | Default `minima` theme |
| CMS | Sveltia CMS | Replaced Decap CMS — see CMS decision section |
| Theme | minima ~2.5 | Classic skin, no cosmetic customisation yet |
| Content storage | GitHub repo | Every post is a Git commit |
| Images | Stored in Git repo (`/uploads`) | Fine at personal blog scale |
| Hosting (GitHub Pages blogs) | GitHub Pages | Free, push-to-deploy |
| Hosting (CF blog) | Cloudflare Pages | Faster deploys, CDN, built-in analytics |
| Auth (GitHub Pages blogs) | Netlify Identity | Free Netlify account required for blog owner |
| Auth (CF blog, front door) | Cloudflare Access | Email/Google login; blocks non-owners from `/admin` |
| Auth (CF blog, CMS) | GitHub OAuth PKCE | Lets the CMS commit posts to the GitHub repo |

---

## Where the Files Are

All three builds live in Paul's agent home directory:

```
/tasklet/agent/home/
├── donaldfromdunedin/           ← GitHub Pages blog build
├── donaldfromdunedin.zip        ← Downloadable zip of above
├── myaudioprojects/             ← GitHub Pages blog build
├── myaudioprojects.zip          ← Downloadable zip of above
├── myaudioprojects-cf/          ← Cloudflare Pages blog build (current focus)
├── myaudioprojects-cf.zip       ← Downloadable zip of above
├── SPEC.md                      ← Canonical spec file (keep up to date)
└── HANDOVER.md                  ← This file
```

None of these have been pushed to GitHub yet. They are template builds — Paul fills in the placeholder values and pushes when setting up a live blog.

---

## Folder Structure (All Three Builds)

Each build follows the same structure:

```
{blogname}/
├── _config.yml         ← Jekyll site settings (title, URL, author, plugins)
├── _posts/             ← Blog posts; managed by CMS
│   └── YYYY-MM-DD-welcome-post.md
├── _layouts/           ← Empty (using minima defaults)
├── _includes/          ← Empty (using minima defaults)
├── assets/
│   └── css/            ← Empty (using minima defaults)
├── uploads/            ← Images uploaded via CMS
├── admin/
│   ├── index.html      ← Loads the Sveltia CMS JavaScript
│   └── config.yml      ← CMS configuration (collections, backend, media)
├── about.md            ← About page (editable via CMS)
├── index.md            ← Homepage (shows post list)
├── Gemfile             ← Jekyll gem dependencies
├── .gitignore          ← Ignores _site/, .bundle/, vendor/
└── README.md           ← Human-readable setup guide for that blog
```

**GitHub Pages builds also have:**
```
└── netlify.toml        ← Netlify config (publish dir, redirect for /admin)
```

**Cloudflare Pages build also has:**
```
└── _headers            ← Cloudflare security headers (X-Frame-Options etc.)
```
No `netlify.toml` in the Cloudflare build.

---

## File-by-File Explanation

### `_config.yml`

Jekyll's main configuration. Key fields:

| Field | What to set |
|---|---|
| `title` | Blog name |
| `description` | Blog tagline |
| `url` | Full domain with https, e.g. `https://myaudioprojects.com` |
| `baseurl` | Leave empty `""` |
| `author.name` | Blog owner's name |
| `theme` | `minima` — do not change |
| `plugins` | `jekyll-feed` and `jekyll-seo-tag` — do not remove |

The GitHub Pages builds also have a `permalink` setting (`/:year/:month/:day/:title/`) and a `defaults` block that sets `layout: post` for all posts. The Cloudflare build omits these — they default to Jekyll's built-ins, which is fine.

The Cloudflare build has a small difference in structure (fewer explicit settings) but is functionally equivalent.

### `admin/index.html`

This file loads the CMS. It is intentionally minimal — just a `<script>` tag:

```html
<script src="https://unpkg.com/@sveltia/cms/dist/sveltia-cms.js"></script>
```

This loads Sveltia CMS from a CDN. The CMS reads `admin/config.yml` to know what to show and how to connect to GitHub.

**Important:** The `noindex` meta tag is present to prevent search engines from indexing the admin panel. Do not remove it.

### `admin/config.yml`

This is the CMS configuration. It has two sections:

**Backend block — how the CMS authenticates and saves:**

For GitHub Pages builds (Netlify Identity):
```yaml
backend:
  name: github
  repo: GITHUB_USERNAME/blogname     # ← REPLACE with actual username/repo
  branch: main
  base_url: https://api.netlify.com
  auth_endpoint: auth
```
The `base_url` and `auth_endpoint` point to Netlify's OAuth proxy. The actual site is NOT on Netlify; Netlify just handles the login flow.

For the Cloudflare Pages build (PKCE):
```yaml
backend:
  name: github
  repo: GITHUB_USERNAME/myaudioprojects   # ← REPLACE with actual username
  branch: main
  auth_type: pkce
  app_id: YOUR_GITHUB_OAUTH_CLIENT_ID     # ← REPLACE with Client ID from GitHub OAuth App
```
PKCE (Proof Key for Code Exchange) is a more modern auth flow that does not require a server-side OAuth proxy. This is why the Cloudflare build has no `netlify.toml` and no Netlify dependency.

**Collections block — what the CMS can edit:**

Both builds define two collections:
- `posts` — maps to the `_posts/` folder; allows creating new posts with title, date, image, body (markdown), and tags
- `pages` — single file collection for `about.md` only; allows editing the About page

**Placeholders that must be replaced before deploying:**
- `GITHUB_USERNAME` → actual GitHub username (Paul's during dev, blog owner's after transfer)
- `YOUR_GITHUB_OAUTH_CLIENT_ID` → Client ID from a GitHub OAuth App (Cloudflare build only)

### `Gemfile`

Specifies Jekyll dependencies. Contents:
```ruby
gem "jekyll", "~> 4.3"
gem "minima", "~> 2.5"
# plugins: jekyll-feed, jekyll-seo-tag
```
No changes needed here. `Gemfile.lock` is generated on first `bundle install` and should be committed.

### `netlify.toml` (GitHub Pages builds only)

```toml
[build]
  publish = "_site"
  command = "jekyll build"

[[redirects]]
  from = "/admin"
  to = "/admin/index.html"
  status = 200
```
Netlify uses this for its own build process (even though it doesn't host the site). The redirect ensures `/admin` resolves correctly. This file is NOT present in the Cloudflare build — Cloudflare handles routing differently.

### `_headers` (Cloudflare Pages build only)

Sets security headers on all responses:
```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: no-referrer-when-downgrade

/admin/*
  X-Robots-Tag: noindex
```
`X-Frame-Options: DENY` prevents the site from being embedded in iframes (clickjacking protection). The `/admin/*` rule adds a robots noindex header on top of the meta tag already in `admin/index.html` for belt-and-braces coverage.

---

## Auth — Detailed Explanation

### GitHub Pages builds — Netlify Identity

Two-part flow:

1. **Netlify Identity** handles who can log into `/admin`. The blog owner creates a Netlify account, links it to the GitHub repo, enables Identity with invite-only registration, enables GitHub as an external provider, and enables Git Gateway. Paul invites the blog owner's email. They accept the invite and can log in with their GitHub account.

2. **Git Gateway** (a Netlify service) acts as a proxy between the CMS and the GitHub repo. When the blog owner publishes a post, Git Gateway makes a commit to the repo on their behalf. This means the blog owner's GitHub account is used for the OAuth, but the commit itself goes through Netlify's server.

**What the blog owner needs:**
- A Netlify account (free)
- A GitHub account (used for OAuth login)

**Limitation:** The blog owner is exposed to Netlify (an extra third-party service). If Netlify changes its free tier, this auth layer could break.

### Cloudflare Pages build — Two layers

This build has a cleaner, more modern auth setup:

**Layer 1 — Cloudflare Access (front door):**
- Cloudflare intercepts all requests to `myaudioprojects.com/admin`
- Presents a login page before the request ever reaches the site
- Sends a one-time login link to the allowed email address
- Only the blog owner's email(s) can get through
- This is configured in Cloudflare Zero Trust → Access → Applications
- Free up to 50 users

**Layer 2 — GitHub OAuth PKCE (CMS saves):**
- Once through the Cloudflare front door, the CMS itself prompts for GitHub login
- Uses PKCE flow — no server needed, all happens client-side
- The CMS gets a GitHub token scoped to the repo
- When the blog owner publishes, the CMS commits directly to GitHub using that token
- Configured via a GitHub OAuth App (Settings → Developer settings → OAuth Apps)
- The Client ID goes into `admin/config.yml` as `app_id`
- The callback URL must be exactly `https://myaudioprojects.com/admin`

**What the blog owner needs:**
- A Cloudflare account (free)
- A GitHub account (used for PKCE OAuth)
- No Netlify account

**Why two layers?** They do different things. Cloudflare Access controls WHO can reach the admin page at all. GitHub OAuth controls HOW the CMS writes content to the repo. One cannot replace the other. GitHub cannot be removed without replacing the entire storage layer (the content would have to go somewhere other than a GitHub repo).

---

## CMS Decision — Sveltia CMS

**All builds use Sveltia CMS.** Decap CMS was the original choice and was replaced.

### Why Decap CMS was dropped
- Largely abandoned — codebase dating to 2016, stagnating
- Unpatched security vulnerabilities including a known XSS flaw
- Bug reports go unanswered, no public roadmap
- The project shows no meaningful maintenance activity

### Why Sveltia CMS was chosen
- Complete rewrite of Decap CMS (not a fork — no legacy code)
- Drop-in replacement: identical config file format, minimal migration effort
- Actively maintained — bugs fixed within 24 hours
- Better editor: mobile support, dark mode, built-in image optimiser, faster load
- Free, no external account required, no cloud dependency

### Other CMS options evaluated and ruled out

| CMS | Reason ruled out |
|---|---|
| **Tina CMS** | Better visual editor but requires TinaCloud backend; 100MB asset cap on free tier is a concern for an audio blog with images; dependency on TinaCloud adds a point of failure |
| **Keystatic** | No meaningful Jekyll support — built for JS frameworks (Astro, Next.js) |
| **Static CMS** | Discontinued September 2024 |
| **Publii** | Desktop app only — can't edit from multiple devices or browsers |
| **CloudCannon** | $49/month minimum — completely wrong price point for personal blogs |
| **Decap CMS** | See above — abandoned with unpatched security issues |

Do not revisit these unless circumstances have materially changed (e.g., Tina CMS dropped its asset cap, CloudCannon launched a free tier).

---

## Comments Research (June 2026)

A conversation was had about adding reader comments to the Cloudflare Pages blog. No decision has been made yet. Here is the full research summary:

### Options evaluated

**Giscus** — Free, no server, backed by GitHub Discussions. Rejected because readers must have a GitHub account to comment. My Audio Projects is a general-interest blog, not a developer audience. This barrier is too high.

**Utterances** — Similar to Giscus but backed by GitHub Issues and less featureful. Same reader-requires-GitHub problem. Ruled out.

**Cusdis** — Lightweight (5kb), privacy-focused, moderation-first, anonymous comments (name + email only, no account needed), no ads, no tracking. Free tier: 100 approved comments/month, 10 quick-approves/month. Pro: $12/year (unlimited). Known issues: no CAPTCHA (spam goes to queue for manual deletion), no nested replies, no reader notifications, intermittent CORS rendering bug. The CORS bug is on Cusdis' CDN and cannot be fixed by setting Cloudflare `_headers` — those only control headers on responses from your own site. The Cusdis script is loaded from Cusdis' CDN, outside your control.

**Disqus** — Easy but rejected: shows ads on free tier, tracks users across the web, privacy is poor.

**Hyvor Talk** — $5/month. Anonymous-friendly, no ads, no tracking, nested replies, reader notifications. More polished than Cusdis. Worth considering if Cusdis' limitations are a problem.

**Remark42** — Best features (anonymous comments, Google/GitHub login, admin moderation, spam filtering, email notifications). Requires a Docker container on a server (~$5/month on Railway or Fly.io). Adds server management overhead. Probably overkill for a personal blog.

### Current status
No comments system has been implemented. The topic is open. Paul's workflow rule applies: review options and get sign-off before writing any code.

---

## Development Conventions — Read These Carefully

Paul has explicit rules for how the agent must behave. These are not suggestions.

1. **Review all design decisions before making changes.** Read the current files, state the plan, get sign-off, then make changes. Do not skip this step even for small changes.

2. **Do not write code unless Paul has approved the approach.** Research and planning happen first. Code comes after explicit approval.

3. **Keep SPEC.md up to date.** When decisions are made or the stack changes, update `/tasklet/agent/home/SPEC.md`. It is the canonical source of truth for the project.

4. **Ask one follow-up question at a time.** Don't pepper Paul with multiple questions. Pick the most important blocker and ask that.

5. **Paul works ad-hoc.** There are no automated triggers or scheduled tasks. This is direct build work in response to Paul's requests.

---

## Setup Instructions Per Blog

### Donald from Dunedin (GitHub Pages)

Full instructions are in `/tasklet/agent/home/donaldfromdunedin/README.md`. Summary:

1. Donald creates a GitHub account
2. Paul pushes repo to his own GitHub
3. Update `admin/config.yml` → set `repo: GITHUB_USERNAME/donaldfromdunedin`
4. Enable GitHub Pages in repo settings (source: main branch, root folder)
5. Connect custom domain `donaldfromdunedin.com` via DNS (A records to GitHub IPs, CNAME www to GitHub)
6. Set up Netlify: connect repo, enable Identity (invite-only, GitHub provider, Git Gateway), invite Donald's email
7. Transfer repo to Donald's GitHub account
8. Update `admin/config.yml` again with Donald's username
9. Update Netlify to point to new repo location

GitHub Pages IP addresses for DNS:
- `185.199.108.153`
- `185.199.109.153`
- `185.199.110.153`
- `185.199.111.153`

### My Audio Projects (GitHub Pages)

Identical process to Donald from Dunedin. Full instructions in `/tasklet/agent/home/myaudioprojects/README.md`.

### My Audio Projects (Cloudflare Pages)

Full instructions in `/tasklet/agent/home/myaudioprojects-cf/README.md`. Summary:

1. Push repo to GitHub (public repo)
2. Create GitHub OAuth App: Settings → Developer settings → OAuth Apps → New. Homepage URL: `https://myaudioprojects.com`. Callback URL: `https://myaudioprojects.com/admin`. Copy the Client ID.
3. Update `admin/config.yml`: set `repo` and `app_id`
4. Connect to Cloudflare Pages: Workers & Pages → Create → Pages → Connect to Git. Build command: `bundle exec jekyll build`. Output dir: `_site`.
5. Connect custom domain in Cloudflare Pages settings
6. Set up Cloudflare Access: Zero Trust → Access → Applications → Add → Self-hosted. Domain: `myaudioprojects.com`. Path: `admin`. Policy: allow specific email(s).
7. Test: visit `/admin`, receive Cloudflare email link, click through, log in with GitHub in CMS

---

## Known Issues and Open Items

| Item | Status |
|---|---|
| Comments system for myaudioprojects-cf | Not implemented. Research done. Options: Cusdis (free, CORS risk), Hyvor Talk ($5/mo), Remark42 (self-hosted). No decision made. |
| Cosmetic theme customisation | Deferred. All blogs use default minima theme with no visual changes. |
| Tags page | Not implemented. Tags can be added to posts via CMS but there is no tag archive page. |
| myaudioprojects-cf `_config.yml` | Has `email: your-email@example.com` placeholder — should be removed or set before deploying. |
| All `admin/config.yml` files | Contain `GITHUB_USERNAME` and (for CF build) `YOUR_GITHUB_OAUTH_CLIENT_ID` placeholders. Must be replaced at deploy time. |

---

## Checking for Sveltia CMS Updates

The CMS is loaded via CDN from unpkg:
```
https://unpkg.com/@sveltia/cms/dist/sveltia-cms.js
```

This always serves the latest version published to npm. No manual update needed — it updates automatically when the page loads. If you need to pin to a specific version (for stability), change the URL to:
```
https://unpkg.com/@sveltia/cms@X.Y.Z/dist/sveltia-cms.js
```

Check current version at: https://www.npmjs.com/package/@sveltia/cms

---

## Running a Blog Locally (for Testing)

Requirements: Ruby, Bundler, Jekyll.

```bash
cd /tasklet/agent/home/myaudioprojects-cf
bundle install
bundle exec jekyll serve
```

Site will be available at `http://localhost:4000`. The CMS admin panel will not be functional locally without configuring a local OAuth flow — for local testing, just inspect the rendered HTML/CSS output.

---

## Key External Links

| Resource | URL |
|---|---|
| Sveltia CMS docs | https://github.com/sveltia/sveltia-cms |
| Sveltia CMS npm | https://www.npmjs.com/package/@sveltia/cms |
| Jekyll docs | https://jekyllrb.com/docs/ |
| minima theme | https://github.com/jekyll/minima |
| Cloudflare Pages docs | https://developers.cloudflare.com/pages/ |
| Cloudflare Access docs | https://developers.cloudflare.com/cloudflare-one/applications/configure-apps/self-hosted-apps/ |
| GitHub OAuth Apps | https://github.com/settings/developers |
| Netlify Identity docs | https://docs.netlify.com/visitor-access/identity/ |

---

## Summary of Current State

All three builds are complete and zipped. No build has been deployed to a live domain yet — they are all local template builds awaiting Paul's deployment.

**Hosting strategy standardised on GitHub Pages (June 2026)** — the Cloudflare Pages build (`myaudioprojects-cf`) has been retired. See the Strategy Update section at the top of this document.

The canonical audio blog build is now `myaudioprojects/` (GitHub Pages + Netlify Identity). The deployment checklist is in the Strategy Update section above.

The key open question is whether to add a comments system to the audio blog. Research was done (Giscus, Utterances, Cusdis, Disqus, Hyvor Talk, Remark42) — see Comments Research section above. No decision has been made. That decision requires Paul's sign-off before any code is written.

Update SPEC.md after any significant changes.

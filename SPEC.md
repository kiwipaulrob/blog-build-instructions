# Blog Build Specifications

## What We're Building

Simple personal blogs — "a cheap dumbed down WordPress."
Day-to-day use: visit `/admin`, log in, write, publish. No GitHub or code exposure for the blog owner.

---

## Stack

| Layer | Choice | Notes |
|---|---|---|
| **Static site generator** | Jekyll | Default `minima` theme |
| **CMS** | Sveltia CMS | Replaces Decap CMS — see CMS decision below |
| **Hosting** | GitHub Pages | Standard for all blogs — see hosting decision below |
| **Auth** | Netlify Identity | Requires free Netlify account for blog owner |
| **Content store** | GitHub repo | Every post/edit is a Git commit |
| **Images** | Stored in Git repo | Fine for personal blog scale |

---

## Hosting Decision — GitHub Pages Standard

**June 2026: Cloudflare Pages retired. GitHub Pages is the standard for all blogs.**

### Why Cloudflare Pages was dropped

During development of a sister project (heargodandevil.com — a generated static music site), a critical constraint was discovered: **Cloudflare Pages has a 25 MB artifact limit**. Git history from iterative builds caused pack files to exceed this limit, breaking deployments entirely.

While a text blog grows slower than a generated music site, the constraint is architectural — it affects any Git-backed project with meaningful history. Combined with other issues, this tipped the decision:

| Issue | Detail |
|---|---|
| **25 MB artifact limit** | Cloudflare Pages rejects pack files over 25 MB — will eventually bite any iterative project |
| **Dual auth complexity** | CF build required two separate auth layers (Cloudflare Access + GitHub OAuth PKCE) — more moving parts, harder to hand over to a non-technical blog owner |
| **Stack inconsistency** | Two of three blogs were already on GitHub Pages; no strong reason to maintain a divergent stack |
| **Thin advantage** | The only real CF win was Cloudflare Access (email/Google login). But GitHub OAuth was still required underneath it for saving posts — the blog owner still needed a GitHub account regardless. |

### Why GitHub Pages works

- No artifact size check — only enforces ~1 GB per repo (blogs are <10 MB)
- Designed for Git-backed projects; handles push-to-deploy correctly
- Native `gh-pages` / main-branch deploy workflow, no extra tooling
- Custom domain support (A records + CNAME)
- Free tier with unlimited bandwidth
- Proven across all three blog builds and sister projects

---

## CMS Decision — Sveltia CMS

**Decap CMS is replaced by Sveltia CMS in all builds.**

### Why not Decap CMS
- Largely abandoned — stagnating codebase dating to 2016
- Unpatched security vulnerabilities including a known XSS flaw
- Bug reports go unanswered, no public roadmap

### Why Sveltia CMS
- Complete rewrite of Decap CMS — not a fork, no legacy debt
- Drop-in replacement: same config file format, minimal migration effort
- Actively maintained — bugs fixed within 24 hours
- Better editor: mobile support, dark mode, built-in image optimiser, faster
- Free, no external account needed

### Other options evaluated and ruled out

| CMS | Ruling |
|---|---|
| **Tina CMS** | Better visual editor but requires TinaCloud backend; 100MB asset cap on free tier is a concern for My Audio Projects |
| **Keystatic** | No meaningful Jekyll support — JS framework only |
| **Static CMS** | Discontinued September 2024 |
| **Publii** | Desktop app only — can't edit from multiple devices |
| **CloudCannon** | $49/month minimum — wrong price point for personal blogs |

---

## Blogs Built

### Donald from Dunedin
- **Path:** `/agent/home/donaldfromdunedin/`
- **Zip:** `/agent/home/donaldfromdunedin.zip`
- **Hosting:** GitHub Pages
- **Domain:** donaldfromdunedin.com
- **Auth:** Netlify Identity
- **Status:** Complete (built with Decap CMS — needs Sveltia CMS swap)

### My Audio Projects (GitHub Pages)
- **Path:** `/agent/home/myaudioprojects/`
- **Zip:** `/agent/home/myaudioprojects.zip`
- **Hosting:** GitHub Pages
- **Domain:** myaudioprojects.com
- **Auth:** Netlify Identity
- **Status:** Complete (built with Decap CMS — needs Sveltia CMS swap)

### My Audio Projects (Cloudflare Pages) — RETIRED
- **Path:** `/agent/home/myaudioprojects-cf/`
- **Zip:** `/agent/home/myaudioprojects-cf.zip`
- **Status:** Retired June 2026. Kept locally for reference only. Do not deploy. Use `myaudioprojects/` (GitHub Pages) instead. See hosting decision above.

---

## Workflow

Ad-hoc build work. No automated triggers.

**Review all design decisions before making changes** — do not write code unless the user has approved the approach.

---

## Development Notes

- User develops on their own GitHub account, then transfers repo to the blog owner
- Blog owners need a GitHub account only for the GitHub OAuth layer (saving posts)
- Cloudflare Access free tier supports up to 50 users — more than sufficient
- Theme is default `minima` — cosmetic changes deferred

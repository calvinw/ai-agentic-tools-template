---
name: github-pages-setup
description: >
  Set up GitHub Pages for a vanilla HTML + Tailwind CSS (Stitch) project repo.
  Use this skill whenever the user wants to publish their site to GitHub Pages,
  enable Pages on their repo, configure the branch/folder settings, or get their
  Stitch project live on the web. Trigger on phrases like "publish to GitHub Pages",
  "set up Pages", "get my site live", "deploy to Pages", "enable GitHub Pages",
  or any time the user has a Stitch/Tailwind HTML project and wants a public URL.
---

# GitHub Pages Setup (Vanilla HTML / Stitch + Tailwind)

## What this skill does

Sets up GitHub Pages for a repo containing a Stitch-exported vanilla HTML project
with Tailwind CSS. The repo structure looks like:

```
repo-root/
├── index.html
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
└── (other .html files, optional)
```

Deployment target: **main branch, root /** (no build step, no gh-pages branch).

---

## Step 1 — Verify the repo is on GitHub with a remote

```bash
git remote -v
```

If there's no remote, the user needs to push to GitHub first:

```bash
gh repo create <repo-name> --public --source=. --remote=origin --push
```

If the remote exists but hasn't been pushed yet:

```bash
git add -A && git commit -m "initial commit" && git push -u origin main
```

---

## Step 2 — Enable GitHub Pages via GitHub CLI

```bash
gh api \
  --method POST \
  -H "Accept: application/vnd.github+json" \
  repos/{owner}/{repo}/pages \
  -f source='{"branch":"main","path":"/"}'
```

To get `owner` and `repo` from the current directory:

```bash
gh repo view --json owner,name
```

So a complete one-liner:

```bash
OWNER=$(gh repo view --json owner -q .owner.login)
REPO=$(gh repo view --json name -q .name)
gh api \
  --method POST \
  -H "Accept: application/vnd.github+json" \
  repos/$OWNER/$REPO/pages \
  -f source='{"branch":"main","path":"/"}'
```

If Pages is already enabled (409 error), use PUT instead of POST to update settings:

```bash
gh api \
  --method PUT \
  -H "Accept: application/vnd.github+json" \
  repos/$OWNER/$REPO/pages \
  -f source='{"branch":"main","path":"/"}'
```

---

## Step 3 — Get the Pages URL

```bash
gh api repos/$OWNER/$REPO/pages --jq '.html_url'
```

The URL will be in the form:
`https://<owner>.github.io/<repo>/`

Tell the user this URL and note that it may take **1–2 minutes** to go live after
the first enable.

---

## Step 4 — Check for asset path issues

For a Stitch project served from a repo subdirectory URL
(`https://owner.github.io/repo-name/`), asset paths in `index.html` must be
**relative**, not absolute.

Check for absolute paths:

```bash
grep -n 'href="/' index.html | head -20
grep -n 'src="/' index.html | head -20
```

If any results appear, those paths will break on Pages. Fix by making them relative:
- `href="/assets/css/style.css"` → `href="assets/css/style.css"`
- `src="/assets/js/main.js"` → `src="assets/js/main.js"`

Stitch exports typically use relative paths by default, so this is usually fine —
but always worth a quick check.

---

## Step 5 — Confirm it's live

```bash
gh api repos/$OWNER/$REPO/pages --jq '{status: .status, url: .html_url}'
```

`status` will be `"built"` when live. If it's still `"queued"` or `"building"`,
wait 60 seconds and check again.

You can also open it directly:

```bash
gh browse --no-browser 2>/dev/null || echo "Open: $(gh api repos/$OWNER/$REPO/pages --jq '.html_url')"
```

---

## Common issues

| Problem | Fix |
|---|---|
| 409 on POST | Pages already exists — use PUT instead |
| 404 after enable | Wait 1–2 min, check `status` field |
| Blank page / missing styles | Asset paths are absolute — make them relative |
| `gh api` auth error | Run `gh auth status` and re-auth if needed |
| No `index.html` at root | Pages needs index.html in root — move it there |

---

## Quick summary for the user

Once done, tell the user:
1. Their Pages URL
2. That it takes ~1-2 minutes to go live the first time
3. That future pushes to `main` auto-deploy — no extra steps needed

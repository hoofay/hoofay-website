# Deploying hoofay.com — Step by Step

This guide takes you from local development to a live site at **hoofay.com**, for free,
using **GitHub Pages** + **GoDaddy** DNS.

---

## Part 1 — Set Up GitHub

### 1.1 Create the repository

1. Go to [github.com](https://github.com) and sign in.
2. Click **New repository** (the green button, or `+` → `New repository`).
3. Name it exactly: `hoofay-website`
4. Set it to **Public** (required for free GitHub Pages).
5. Leave everything else blank — no README, no .gitignore.
6. Click **Create repository**.

### 1.2 Connect your local project to GitHub

Open a terminal in the `hoofay-website` folder and run:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR-GITHUB-USERNAME/hoofay-website.git
git push -u origin main
```

Replace `YOUR-GITHUB-USERNAME` with your actual GitHub username.

---

## Part 2 — Set Up GitHub Actions (Auto-Deploy)

Every time you push code to GitHub, this will automatically build and publish the site.

### 2.1 Create the workflow file

The file `.github/workflows/deploy.yml` in the project already handles this.
If it doesn't exist, create it:

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

Commit and push this file.

### 2.2 Enable Pages in GitHub

1. Go to your repository on GitHub.
2. Click **Settings** → **Pages** (left sidebar).
3. Under **Source**, select **GitHub Actions**.
4. Save.

Your site will now auto-build on every push. The first build takes ~2 minutes.
You can watch it at: `https://github.com/YOUR-USERNAME/hoofay-website/actions`

At this point, your site is live at:
`https://YOUR-USERNAME.github.io/hoofay-website/`

---

## Part 3 — Connect hoofay.com (GoDaddy DNS)

This makes `hoofay.com` point to your GitHub Pages site.

### 3.1 Add a CNAME file to your project

Create a file called `CNAME` in the `public/` folder with one line:

```
hoofay.com
```

Commit and push this. It tells GitHub Pages which domain to expect.

### 3.2 Update GoDaddy DNS

1. Log in to [godaddy.com](https://godaddy.com).
2. Go to **My Products** → find `hoofay.com` → click **DNS**.
3. You need to add/update these records:

| Type  | Name | Value                 | TTL  |
|-------|------|-----------------------|------|
| A     | @    | 185.199.108.153       | 1hr  |
| A     | @    | 185.199.109.153       | 1hr  |
| A     | @    | 185.199.110.153       | 1hr  |
| A     | @    | 185.199.111.153       | 1hr  |
| CNAME | www  | YOUR-USERNAME.github.io | 1hr |

> **Note:** Delete any existing A records pointing to GoDaddy's parked page first.
> The four IP addresses above are GitHub Pages' servers (correct as of 2025).

4. DNS changes take up to **24–48 hours** to fully propagate, but usually much faster.

### 3.3 Enable HTTPS in GitHub

Once DNS has propagated (hoofay.com resolves to your site):

1. Go to your repo → **Settings** → **Pages**.
2. Under **Custom domain**, type `hoofay.com` and click Save.
3. Tick **Enforce HTTPS** once it appears (may take a few minutes after DNS propagates).

---

## Part 4 — Day-to-Day Workflow

### Making changes

```bash
# 1. Edit files locally
# 2. Test with the dev server:
npm run dev

# 3. When happy, commit and push:
git add .
git commit -m "Add new game: snake"
git push
```

GitHub Actions will automatically rebuild and deploy. Takes ~2 minutes.

### Previewing locally

```bash
npm run dev
# Opens at http://localhost:4321
```

### Building locally to check for errors

```bash
npm run build
# Output goes to /dist — same as what gets deployed
```

---

## Troubleshooting

**Site not updating after push?**
Check the Actions tab in GitHub for build errors.

**Custom domain keeps resetting in GitHub Pages settings?**
Make sure `public/CNAME` file is committed with content `hoofay.com`.

**Getting a 404 on all pages except home?**
The `astro.config.mjs` `site` value must match your domain exactly.

**GoDaddy shows "DNS not propagated"?**
Normal — wait up to 48 hours. Check propagation at [dnschecker.org](https://dnschecker.org).

# Deployment Guide — YOSHIO Lab Website

End-to-end walkthrough: GitHub → Netlify auto-deploy → Identity-protected
admin → custom domain with HTTPS. Total time ≈ 30 minutes. Total cost: 0
(everything fits in the free tier).

## How the pieces fit together

GitHub holds your source code — every push triggers a redeploy. Netlify
hosts the static site, terminates HTTPS, and provides the **Identity** +
**Git Gateway** services that let Decap CMS commit changes back to
GitHub on your behalf. Decap CMS lives at `/admin/` on the deployed
site: anyone can open that URL, but only people you invite through
Netlify Identity can actually log in and save changes.

---

## Step 1 — Push your code to GitHub

### 1A. Create the repo
1. Sign in at [github.com](https://github.com).
2. Click the green **New** button (or visit [github.com/new](https://github.com/new)).
3. Repository name: `yoshio-lab` (or anything you prefer).
4. **Public** is fine. Private also works on Netlify's free tier.
5. **Do not** check "Add a README" — your folder already has one.
6. Click **Create repository**.

### 1B. Push from your local folder

Open Terminal (macOS) / PowerShell (Windows), then:

```bash
cd "/Users/liuchengyang/Claude Code/Web for YOSHIO Lab"

git init
git add .
git commit -m "Initial commit: YOSHIO Lab site + Decap CMS"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/yoshio-lab.git
git push -u origin main
```

The first `git push` will ask for credentials. If GitHub no longer
accepts your password, create a **personal access token**:
GitHub → **Settings → Developer settings → Personal access tokens →
Tokens (classic) → Generate new token (classic)** → tick `repo` scope →
generate → paste the token as the password.

> Prefer a GUI? Install [GitHub Desktop](https://desktop.github.com),
> drag the project folder into its window, click **Publish repository**.
> Same result.

### Subsequent updates
After any local change:

```bash
git add .
git commit -m "Update something"
git push
```

Netlify will redeploy automatically after Step 2.

---

## Step 2 — Connect Netlify for auto-deploy

1. Sign up at [app.netlify.com](https://app.netlify.com) — easiest is **Continue with GitHub**.
2. Click **Add new site → Import an existing project → Deploy with GitHub**.
3. Authorize Netlify if asked, then pick your `yoshio-lab` repo.
4. Build settings:
   - **Branch to deploy:** `main`
   - **Build command:** *(leave empty)*
   - **Publish directory:** *(leave empty, or type `/`)*
5. Click **Deploy site**.

Netlify assigns a random URL like `radiant-pebble-1234.netlify.app`.
Rename it under **Site configuration → Site information → Change site name**
to something meaningful (e.g. `yoshio-lab.netlify.app`).

From this point on, every `git push` to `main` triggers an automatic
rebuild. The **Deploys** tab shows live progress.

---

## Step 3 — Protect `/admin/` with Netlify Identity

The gate that ensures only you can log into the CMS.

### 3A. Enable Identity
Site dashboard → **Integrations → Identity** → **Enable Identity**.

### 3B. Lock down sign-ups
Still in Identity → **Settings and usage**, scroll to **Registration
preferences** → switch to **Invite only**. **This is critical** — otherwise
anyone with the URL can self-register.

### 3C. Enable Git Gateway
Identity → **Services → Git Gateway** → **Enable Git Gateway**. This
creates a token Netlify uses to commit on your behalf, so the CMS can
write to GitHub without you exposing any personal credentials.

### 3D. Invite yourself
Identity → **Identity users → Invite users** → type your email →
**Send invite**. Check your inbox, click the link, set a password.

You are now the only person who can edit content.

### Day-to-day editing workflow
1. Go to `https://yoshio-lab.netlify.app/admin/`
2. Click **Login with Netlify Identity** → enter email + password.
3. Edit any collection (News, Research, Publications, Members, Site).
4. Click **Publish**.
5. Decap commits to GitHub → Netlify rebuilds → live in ~30 s.

> **Security note:** anyone can open `/admin/` and see the login screen,
> but nothing happens without a valid Identity account. Because
> registration is **Invite only**, nobody can create an account
> themselves. If you ever want to revoke access, delete the user under
> Identity → Identity users.

---

## Step 4 — Connect a custom domain

Suppose you've bought `yoshio-lab.com`. You have two paths.

### Option A — Use Netlify DNS (simplest, recommended)

1. Site dashboard → **Domain management → Add a domain you already own** → enter `yoshio-lab.com`.
2. Netlify will offer **Set up Netlify DNS** → click it and follow the prompts.
3. Netlify shows you four nameserver addresses, e.g.:
   ```
   dns1.p07.nsone.net
   dns2.p07.nsone.net
   dns3.p07.nsone.net
   dns4.p07.nsone.net
   ```
4. Log in to your domain registrar (GoDaddy, Namecheap, Google Domains,
   Cloudflare Registrar, etc.) and **replace the default nameservers**
   with the four Netlify ones.
5. Wait 5 minutes to ~24 hours for DNS to propagate. You can check
   propagation at [dnschecker.org](https://dnschecker.org).
6. Netlify automatically provisions a **Let's Encrypt SSL certificate**
   once it sees the DNS pointing at it. HTTPS is on by default.

### Option B — Keep DNS at your registrar, add records by hand

If you prefer not to transfer DNS hosting (e.g. you have other records
for email, MX), log in to your registrar and add:

| Type   | Host / Name | Value                          | TTL   |
|--------|-------------|--------------------------------|-------|
| A      | `@`         | `75.2.60.5`                    | 3600  |
| CNAME  | `www`       | `yoshio-lab.netlify.app.`      | 3600  |

(`75.2.60.5` is Netlify's apex load-balancer IP. If your registrar
supports `ALIAS` / `ANAME` records for the apex, use
`apex-loadbalancer.netlify.com.` instead — it's more resilient than a
hard-coded IP.)

Then back on Netlify: **Domain management → Add custom domain →
`yoshio-lab.com`** → click **Verify**.

### Force HTTPS + canonical redirect
Once Netlify verifies DNS (5–15 min):

1. **Domain management → HTTPS** → **Provision certificate** (usually automatic).
2. Toggle **Force HTTPS** on.
3. Pick your **primary domain** (e.g. `yoshio-lab.com`); Netlify auto-301-redirects `www.yoshio-lab.com` to it.

After this, `http://yoshio-lab.com` and `www.yoshio-lab.com` all funnel into `https://yoshio-lab.com`.

---

## Troubleshooting

**CMS shows "Error: Could not authenticate"**
You skipped Step 3C (Git Gateway), or the `branch` in `admin/config.yml`
doesn't match your repo's default branch. Currently it's set to `main`.

**Site updates but `/admin/` preview doesn't reflect changes**
Browser is caching the YAML. Hard-reload with **Cmd+Shift+R** (macOS) or
**Ctrl+Shift+R** (Windows).

**`_data/*.yml` returns 404 in DevTools**
Make sure `.nojekyll` is committed at the repo root. Netlify usually
doesn't need this, but it's a safety net.

**Image upload fails in CMS**
Confirm `admin/config.yml` has `media_folder: "assets/images"` and the
`assets/images/` folder is committed (the `.gitkeep` keeps it tracked
when empty).

**"Build failed" on Netlify**
Open the **Deploys** tab → click the failed deploy → read the log. For
this project there's no build step, so any failure is almost always a
typo in `admin/config.yml` YAML (indentation, missing colon, etc.). Fix,
commit, push.

---

## Quick reference

| What                         | Where                                               |
|------------------------------|-----------------------------------------------------|
| Public site                  | `https://yoshio-lab.netlify.app` (or custom domain) |
| Admin / CMS login            | `https://yoshio-lab.netlify.app/admin/`             |
| Source code                  | `https://github.com/YOUR-USERNAME/yoshio-lab`       |
| Build & deploy logs          | Netlify dashboard → **Deploys**                     |
| Invite / remove editors      | Netlify dashboard → **Identity → Identity users**   |
| DNS / domain settings        | Netlify dashboard → **Domain management**           |

That's all. Once the four steps are done, you'll never have to touch
code again — just `/admin/` to edit, **Publish** to ship.

# YOSHIO Lab Website + Decap CMS

A bilingual (EN / JA), responsive single-page site for the YOSHIO Lab
(Molecular Mechatronics Group, NIMS), with a built-in Decap CMS admin
panel so you can edit News, Research, Publications, Members, Hero/About/Contact
and upload images **without touching any code**.

---

## 1. File / folder structure

Everything you need to commit to your GitHub repository:

```
├── index.html              ← Main site (loads YAML data at runtime)
├── .nojekyll               ← Tells GitHub Pages NOT to run Jekyll on _data/
├── README.md               ← This file
│
├── admin/                  ← Decap CMS admin panel
│   ├── index.html          ← Loads the CMS + custom previews
│   └── config.yml          ← Field configuration for every section
│
├── _data/                  ← Editable content (YAML)
│   ├── site.yml            ← Hero, About, Stats, Contact, brand
│   ├── news.yml            ← Timeline news items
│   ├── research.yml        ← Research themes
│   ├── publications.yml    ← Publication list (auto-grouped by year)
│   └── members.yml         ← Members + Join-Us text
│
└── assets/
    └── images/             ← All uploaded images land here
        └── .gitkeep
```

> **Don't rename `_data/` or `admin/`.** `admin/config.yml` and the JS in
> `index.html` reference these paths directly.

The four color variants (`variant-*.html`) are optional — keep them for
reference or delete them.

---

## 2. How to deploy (two paths)

### Option A — Netlify (easiest, recommended)

1. Push the folder to a GitHub repo.
2. Go to **app.netlify.com** → "Add new site" → "Import from GitHub" →
   select your repo. Build command: leave empty. Publish directory: `/`.
3. In Netlify dashboard → **Site settings → Identity** → **Enable Identity**.
4. **Identity → Registration preferences** → set to **Invite only**, then
   click **Invite users** and invite yourself.
5. **Identity → Services → Git Gateway** → click **Enable Git Gateway**.
6. Open `https://YOUR-SITE.netlify.app/admin/`, accept the email invite,
   set a password — you're in.

The default `admin/config.yml` is already configured for this path
(`backend: git-gateway`). No code changes needed.

### Option B — GitHub Pages

1. Push to a GitHub repo, enable **Settings → Pages → Deploy from branch (main)**.
2. The static site will work immediately at `https://USERNAME.github.io/REPO/`.
3. **For CMS auth on GitHub Pages**, you must register a GitHub OAuth app
   and host an OAuth provider (Decap provides one — see
   [Decap GitHub backend docs](https://decapcms.org/docs/github-backend/)).
   Then edit `admin/config.yml` and switch to:

   ```yaml
   backend:
     name: github
     repo: YOUR-USERNAME/YOUR-REPO
     branch: main
     base_url: https://YOUR-OAUTH-PROVIDER.example.com
   ```

   For most users, **Option A is much simpler**.

---

## 3. How to edit content

Once deployed:

1. Visit **`https://YOUR-SITE/admin/`** and log in.
2. Pick a collection on the left:
   - **Site** — hero, about, stats, contact, brand
   - **News** — add/remove/reorder news items (timeline)
   - **Research** — six (or any number) research-theme cards with images
   - **Publications** — fill the form for each paper; year is auto-grouped
   - **Members** — PI, postdoc, students, alumni; upload a photo per person
3. Edit any field. Right-side pane shows a **live preview** styled to
   match the public site.
4. Click **Publish** → Decap commits the change to GitHub → Netlify
   redeploys → the public site updates within ~30 s.

### Upload images
Every image field has an upload button. Files go to `assets/images/` and
the path is automatically inserted into the YAML.

### Bilingual content
Each text field has an `_en` and `_ja` (or "EN" / "JA") version. The
public site uses the EN/JP toggle in the nav bar to switch between them.

### Adding a publication
Members collection → **Publications → Add** →
* Year (e.g. `2025`)
* Authors (`**Bold lab members**` — `**M. Yoshio**` will render as bold)
* Title
* Journal
* Volume / pages
* DOI

The site automatically groups by year and shows a year-tab on the front-end.

---

## 4. Local testing

Decap needs a real HTTP server (not `file://`) because it `fetch()`es
YAML.

```bash
# Easiest — Python built-in
python3 -m http.server 8000
# then open http://localhost:8000/
```

For the CMS admin locally:

```bash
# Run a local backend that writes to your working tree
npx decap-server &
python3 -m http.server 8000
# then open http://localhost:8000/admin/
```

(`local_backend: true` is already set in `admin/config.yml`.)

---

## 5. Quick checklist before going live

- [ ] Repo on GitHub with all files above
- [ ] `.nojekyll` present at the root
- [ ] Netlify site connected to repo
- [ ] Netlify Identity + Git Gateway enabled
- [ ] You've received the invite email and set your CMS password
- [ ] `/admin/` opens and the right-pane preview shows your edits

---

## 6. Tech stack

- HTML + Tailwind CSS (CDN) — single-file site, no build step
- Vanilla JavaScript — fetches and renders YAML at runtime
- [js-yaml](https://github.com/nodeca/js-yaml) — YAML parser in the browser
- [Decap CMS](https://decapcms.org/) — Git-based content management
- Netlify (recommended host) + GitHub (source of truth)

Designed with care · Built with HTML & Tailwind CSS.

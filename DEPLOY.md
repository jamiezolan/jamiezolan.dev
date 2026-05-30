# Deploying this site

Two stages. **Stage 1** gets you live for free in about ten minutes. **Stage 2** is optional polish that puts it on `jamiezolan.dev`. Do Stage 1 now; Stage 2 whenever.

## Stage 1 — Ship on GitHub Pages (free)

1. Create a new GitHub repo. Naming it `jamiezolan.dev` keeps things tidy, but any name works — the repo name becomes part of the temporary URL.
2. Push this project:
   ```bash
   git init
   git add .
   git commit -m "Pudgy docs site"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo>.git
   git push -u origin main
   ```
3. In the repo: **Settings → Pages → Build and deployment → Source → GitHub Actions**. The workflow in `.github/workflows/deploy.yml` handles the build and publish.
4. The push triggers a build. When it's green in the **Actions** tab, your site is live at:
   `https://<your-username>.github.io/<repo>/`

That URL is perfectly fine to put in job applications today. Everything below is optional.

> Remember to update `repo_url` / `repo_name` in `mkdocs.yml` with your real GitHub handle so the "edit this page" links work.

## Stage 2 — Put it on jamiezolan.dev

### a. Register the domain
Buy `jamiezolan.dev` at a registrar that gives you real DNS control — **Cloudflare** or **Porkbun** are both good and cheap. Confirm availability at checkout; if it's taken, `jamie-zolan.dev` or `zolan.dev` are clean fallbacks.

> **`.dev` quirk:** the entire `.dev` TLD is on the browser HTTPS-always (HSTS preload) list, so browsers refuse plain HTTP. This is fine — GitHub Pages issues a free certificate — but the site will look broken until that cert finishes provisioning after you set the custom domain. Just wait it out; it's not a misconfiguration.

### b. Tell GitHub the domain
Repo → **Settings → Pages → Custom domain** → enter `jamiezolan.dev` → **Save**. (A `CNAME` file with this value already ships in `docs/`, so every build keeps the domain set.) Once GitHub finishes issuing the TLS certificate, tick **Enforce HTTPS**.

### c. Add the DNS records
At your registrar's DNS panel, point the apex domain at GitHub Pages. Pick whichever row your provider supports.

**Simplest** — Cloudflare and Porkbun support an apex `ALIAS`/`CNAME`, so it's one record:

| Type | Name | Value |
|---|---|---|
| ALIAS / CNAME | `@` | `<your-username>.github.io` |

On Cloudflare, set this record to **DNS only** (grey cloud, not the orange proxy) so GitHub manages the certificate directly.

**Universal** — works at any provider. Four `A` records and four `AAAA` records on the apex:

| Type | Name | Value |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |

Then add `www` as well (GitHub recommends it, and it's the most stable kind of record):

| Type | Name | Value |
|---|---|---|
| CNAME | `www` | `<your-username>.github.io` |

### d. Verify
```bash
dig jamiezolan.dev +noall +answer -t A
```
The four GitHub IPs should come back. DNS propagation ranges from minutes to a day.

## Then: connect the two halves
On your Journo portfolio at `jamiezolan.com`, add a visible link to `jamiezolan.dev`, and link back from this site's footer. That cross-link is what makes the `.com` / `.dev` split read as one coherent person — portfolio on one, technical work on the other — instead of two stray sites.

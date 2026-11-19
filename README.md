# Florida Trust and Probate — website

RLS FIRM, PLLC d/b/a Florida Trust and Probate.

This repo is the whole website: a single page, `index.html`, served by GitHub
Pages at **https://fltp.law**. Two other domains redirect to it.

| Domain | What it does | DNS hosted at | Registered at |
|---|---|---|---|
| **fltp.law** | Serves the site (GitHub Pages) | Cloudflare, *RLS FIRM* account | NameSilo |
| www.fltp.law | Cloudflare 301 redirect to fltp.law | Cloudflare | — |
| hgreid.com (+ www) | 301 redirect to fltp.law | Cloudflare, *RLS FIRM* account | check (not NameSilo DNS) |
| floridatrustandprobate.com (+ www) | 301 redirect to fltp.law | Cloudflare, *RLS FIRM* account | check |

All three domains also receive email through **Google Workspace**. Their MX,
SPF, DKIM (`google._domainkey`), DMARC (`_dmarc`) and `google-site-verification`
records live in Cloudflare and must not be removed or changed.

---

## Updating the page

1. Edit `index.html`. Everything (styles included) is inside that one file;
   there are no images, scripts, fonts or other dependencies.
2. Commit and push to `master`.
3. GitHub Pages rebuilds automatically; the change is live at https://fltp.law
   within a minute or two. The redirect domains need nothing — they always land
   on whatever fltp.law is serving.

Check the build with `gh api repos/heartlandtitle/website-fltp/pages --jq .status`
(`built` = live) or in the repo's **Actions** tab.

### Content rules (Florida Bar advertising)

This is a law firm advertisement governed by Rules 4-7.11 through 4-7.22 of the
Rules Regulating The Florida Bar. **Do not change wording without the
attorney's approval.** In particular:

1. Board certification must always read "Board Certified in Real Estate Law by
   The Florida Bar" (area + certifying body). Never describe the firm as
   certified, and never apply "certified," "specialist," or "expert" to
   probate, trusts, or estate planning.
2. "One of 404 lawyers ... (as of November 2026)" — the number and date must be
   updated together whenever the Bar's count changes.
3. The Sebring location must keep the "Available by appointment only" label.
4. The footer must keep the d/b/a line and the "Attorney responsible for the
   content of this website" line, legible and conspicuous.
5. Do not add testimonials, reviews, past results, fee statements, stock photos
   of people, or chat/contact widgets without the attorney's review.
6. The phone links use `tel:+14073213911` and work on mobile.

---

## How the hosting works

### GitHub Pages (repo `heartlandtitle/website-fltp`)

- Publishes the root of the `master` branch (classic "deploy from branch"
  Jekyll build).
- Custom domain `fltp.law` is set in **Settings → Pages** and recorded in the
  `CNAME` file. Don't edit or delete `CNAME`.
- "Enforce HTTPS" is on. GitHub issues and renews the certificate itself, which
  only works because the fltp.law web records in Cloudflare are **DNS only**
  (grey cloud) — see below.
- `_config.yml` excludes `README.md` and `CLAUDE.md` from the published site.
  The repo is public, but these notes are not served on fltp.law. Any new
  non-website file should be added to that `exclude` list.

### Cloudflare DNS for fltp.law

| Type | Name | Content | Proxy |
|---|---|---|---|
| A | fltp.law | 185.199.108.153 | DNS only |
| A | fltp.law | 185.199.109.153 | DNS only |
| A | fltp.law | 185.199.110.153 | DNS only |
| A | fltp.law | 185.199.111.153 | DNS only |
| CNAME | www | heartlandtitle.github.io | **Proxied** (see below) |
| MX ×5 | fltp.law | aspmx.l.google.com (1), alt1/alt2.aspmx.l.google.com (5), alt3/alt4.aspmx.l.google.com (10) | — |
| TXT | fltp.law | `v=spf1 include:_spf.google.com ~all` | — |
| TXT | fltp.law | `google-site-verification=…` | — |
| TXT | _dmarc | `v=DMARC1; p=none` | — |
| TXT | google._domainkey | Google Workspace DKIM key | — |

The four A records are GitHub Pages' published IPs. **Keep them grey-cloud.**
Turning on the Cloudflare proxy (orange cloud) breaks GitHub's certificate
renewal.

`www` is the exception: GitHub's certificate only covers `fltp.law`, so
`https://www.fltp.law` used to fail with a certificate error. Instead, the `www`
CNAME is proxied and a Cloudflare Redirect Rule on the fltp.law zone ("Redirect
www to apex", `http.host eq "www.fltp.law"`) sends it to
`https://fltp.law` + path with a 301, preserving the query string. Cloudflare
supplies the `www` certificate; GitHub never sees `www` traffic.

### Redirect domains (hgreid.com, floridatrustandprobate.com)

Each zone has:

- `A @` and `A www` → `192.0.2.1`, **proxied** (orange cloud). The IP is a
  documentation placeholder; traffic never reaches it because Cloudflare
  answers first. The records just need to exist and be proxied so Cloudflare
  receives the request and has a certificate for it.
- A **Redirect Rule** (Rules → Redirect Rules, phase
  `http_request_dynamic_redirect`) named "Redirect to fltp.law":
  - When: `(http.host in {"<domain>" "www.<domain>"})`
  - Then: 301 to `concat("https://fltp.law", http.request.uri.path)`,
    preserve query string.
  - So `www.hgreid.com/some/page?x=1` → `https://fltp.law/some/page?x=1`.

hgreid.com previously hosted a Wix site; that was replaced by the redirect in
November 2026. Its other records (Google `mail`/`calendar`/`docs`/… CNAMEs, a
`*` wildcard A record, and two leftover `NS` records pointing at worldnic.com)
were left untouched.

### Registrars and renewals

- **fltp.law is registered at NameSilo** and only its nameservers point to
  Cloudflare (`crystal.ns.cloudflare.com`, `rommy.ns.cloudflare.com`). It
  **cannot** be moved to Cloudflare Registrar — Cloudflare doesn't support the
  `.law` extension. Renew it at NameSilo (it was due early December 2026);
  keep auto-renew on there.
- Changing nameservers or DNSSEC for fltp.law is done in NameSilo; everything
  else (records, redirects) is done in Cloudflare.

---

## Common tasks

**Point another domain at the site.** Add it to the RLS FIRM Cloudflare
account (or use one already there), create the two proxied `192.0.2.1` A
records, and add the same redirect rule with the new hostnames. Keep any
existing MX/TXT email records. Don't try to add it as a second custom domain in
GitHub Pages — a Pages site supports only one.

**Change what fltp.law shows.** Edit `index.html`, commit, push. Nothing in
Cloudflare needs to change.

**Site down / certificate error on fltp.law.** Check
`gh api repos/heartlandtitle/website-fltp/pages` (status, `cname`,
`https_certificate`) and make sure the four fltp.law A records in Cloudflare
are unchanged and grey-cloud. Removing and re-adding the custom domain in
Settings → Pages forces GitHub to re-issue the certificate.

---

## Working on this with Claude Code

Tools Claude can use here:

- **`gh`** (GitHub CLI) — repo, Pages settings and build status.
- **`cf`** (Cloudflare CLI) — DNS records, zones and rules. Log in with
  `cf auth login`. If your login can see more than one Cloudflare account, set
  `CLOUDFLARE_ACCOUNT_ID` to the RLS FIRM account's ID first (find it with
  `cf auth whoami`), otherwise commands fail with "More than one account
  available". Useful commands:
  - `cf dns records list -z fltp.law`
  - `cf rulesets entrypoint get http_request_dynamic_redirect -z hgreid.com`
  - `cf zones list --account-id <id>`

Things to tell Claude (also in `CLAUDE.md`):

- Don't change page wording that falls under the content rules above without
  the attorney's sign-off.
- Never touch the Google Workspace email records.
- Keep the fltp.law apex A records DNS only (only `www` is proxied).
- `rulesets entrypoint update` replaces the whole rule list for that zone —
  read the existing rules first and include them.

# Notes for Claude

Read `README.md` first — it explains the site, the domains, the DNS and the
redirects.

- `index.html` is a Florida Bar–regulated law firm advertisement. Do not change
  its wording unless the user says the attorney approved it, and follow the
  content rules in the README.
- Deploying = committing and pushing to `master` (GitHub Pages). Pushing makes
  changes live immediately, so confirm with the user before pushing.
- Never modify or delete the Google Workspace email records (MX, SPF, DKIM,
  DMARC, site verification) on any of the domains.
- fltp.law's apex A records must stay DNS only (not proxied). The `www`
  CNAME is intentionally proxied; a Cloudflare redirect rule sends it to the
  apex.
- `cf rulesets entrypoint update` replaces all rules in that phase — fetch the
  existing ruleset first and include its rules.
- Any new file that isn't part of the website must be added to `exclude` in
  `_config.yml` so it isn't published on fltp.law.

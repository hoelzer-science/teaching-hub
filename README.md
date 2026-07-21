# teaching-hub

Public landing site for teaching: courses, philosophy, infrastructure,
resources. Deployed to **teaching.hoelzer.science**.

This is the public front door. It links out to the individual course sites
(private, password-protected) and to the open
[teaching-template](https://github.com/hoelzer-science/teaching-template).

## Local

Quarto only — no Python or pixi, since there is no executed code.

```bash
quarto preview        # live at localhost
quarto render         # build into _site/
./check-links.sh _site
```

## Deployment

CI renders, checks links, and deploys to Cloudflare Pages on every push to
`main`. Unlike the course sites there is **no auth worker** — the hub is public.

One-time setup (see the template's README for the detailed Cloudflare walkthrough):

1. Create the Pages project: `wrangler pages project create teaching-hub --production-branch=main`
2. GitHub repository → Settings → Secrets and variables → Actions:
   - secrets: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`
   - variable: `CLOUDFLARE_PROJECT_NAME` (e.g. `teaching-hub`)
3. Add the custom domain `teaching.hoelzer.science` from the Pages project's
   Custom domains tab (do not create the DNS record by hand).

The API token and account ID are the same as for the course projects.

## Before sharing widely

- `philosophy.qmd` and `resources.qmd` carry author notes (HTML comments) —
  personalise before publicising.
- `contact.qmd` uses a personal email; consider an institutional address.
- Course links point at `*.hoelzer.science`; planned courses are listed
  without links until their sites exist.

## Relation to the other repos

```
teaching-hub          public   this repo — the front door
teaching-template     public   reusable course skeleton
<module>              private   actual courses (bioinformatics, ...)
```

Styling in `styles/website.scss` is shared with the template by hand; keep the
palette variables in sync so the sites read as one family.

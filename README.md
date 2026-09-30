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

Setup is done. For reference (see the template's README for the detailed
Cloudflare walkthrough):

- Pages project name is **`teaching`** — `teaching.pages.dev` was already taken,
  so the default subdomain is `teaching-62z.pages.dev`.
- GitHub repository → Settings → Secrets and variables → Actions:
  - secrets: `CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`
  - variable: `CLOUDFLARE_PROJECT_NAME` = `teaching`
- The custom domain was added from the Pages project's Custom domains tab.
  Never create the DNS record by hand.

The API token and account ID are the same as for the course projects.

## Publishing constraint

The hub's framing is employer-agnostic: courses are not presented as "courses
at institution X", and no module codes appear. The About page may name
Martin's employers, as any bio does. See `CLAUDE.md` for the history of this
rule.

## Relation to the other repos

```
teaching-hub          public   this repo — the front door
teaching-template     public   reusable course skeleton
<module>              private   actual courses (bioinformatics, ...)
```

Styling in `styles/website.scss` is shared with the template by hand; keep the
palette variables in sync so the sites read as one family.

# CLAUDE.md — teaching-hub

Permanent onboarding for this repository. Changes only when project knowledge
changes. Current working state lives in `NEXT.md`; session history in
`docs/sessions/`.

## Project overview

The public front door for Martin Hölzer's teaching: philosophy, the open
infrastructure behind the courses, openly licensed resources, an FAQ, contact.
Deployed to **teaching.hoelzer.science**.

Its audience is people **not yet in a course** — prospective students, other
educators, anyone curious about the setup. Material for *enrolled* students
belongs on the individual course sites, behind their password.

## Goals

- Be the stable, citable home for the teaching approach and the open material.
- Point at the reusable template so the infrastructure is genuinely reusable (OER).
- Stay trivially maintainable: no build tooling beyond Quarto, no executed code.

## High-level architecture

Three repositories, all Quarto + Cloudflare Pages:

```
teaching-hub          public    this repo — the front door       teaching.hoelzer.science
teaching-template     public    reusable course skeleton (OER)   teaching-template.hoelzer.science (401)
course-<module>       private   actual courses                   course-<module>.hoelzer.science (401)
<shared-prerequisite> public    material every module needs      first instance: linux.hoelzer.science (live)
```

**Naming rule.** The `course-` prefix marks a password-protected enrolled
module; everything public takes a bare label. Repo name, Pages project name and
subdomain label are always identical, so there is nothing to map. The domain's
first-level labels are a scarce permanent namespace reserved for durable
identities — a future research group should be able to have plain
`bioinformatics`. Nesting under `teaching.` is not possible on the free tier:
Universal SSL covers the apex and `*.hoelzer.science` but not
`*.teaching.hoelzer.science`, so everything lives one level deep.

The fourth kind exists because the first three have **no channel for sharing content**:
cherry-pick sync is infrastructure only, so anything copied per module drifts. Material that
every module needs and none owns — the first is a Linux/bash crash course — gets its own
public repo and site, linked from each module and from `resources.qmd` here.

The hub is the only one of the three that is meant to be **read by the public**.
It has no auth worker, no pixi environment, and no LMS build — deliberately.

## Technology stack

| Piece | Tool |
|---|---|
| Source | Markdown + Quarto (website project) |
| Theme | `cosmo` + `styles/website.scss` |
| CI | GitHub Actions (`.github/workflows/publish.yml`) |
| Hosting | Cloudflare Pages, Direct Upload via `wrangler` |

No Python, no pixi, no lockfile — there is no executed code here. If you find
yourself adding an environment, ask whether the content belongs in a course repo
instead.

## Important directories and files

| Path | What it is |
|---|---|
| `_quarto.yml` | site config, navbar, footer |
| `index.qmd` | landing page and course list |
| `philosophy.qmd` | the teaching philosophy, in Martin's own voice |
| `infrastructure.qmd` | the open stack, for other educators |
| `resources.qmd` | licensing and reusable material |
| `faq.qmd`, `contact.qmd` | self-explanatory |
| `images/` | figures; see the SVG note under Constraints |
| `styles/website.scss` | duplicated by hand with the template — keep palettes in sync |
| `check-links.sh` | every local link must resolve; CI runs it |

## Coding conventions and style

- **Prose is Martin's voice.** Philosophy and index copy were rewritten by him;
  do not "improve" tone unasked. Propose wording, let him decide.
- Hard-wrap `.qmd` prose at roughly 80 columns, matching the existing files.
- British/European spelling ("analysing", "licensed"), en dashes for asides.
- Every figure gets a caption **and** a `fig-alt`.
- No `<!-- AUTHOR NOTE -->` scaffolding in committed files — it shipped once and
  had to be cleaned up.

## Common commands

```bash
quarto preview          # live-reloading site
quarto render           # build into _site/
./check-links.sh _site  # what CI runs; must pass before pushing
```

Deploy status and secrets:

```bash
gh run list -R hoelzer-science/teaching-hub --limit 3
gh secret list -R hoelzer-science/teaching-hub
npx wrangler pages deployment list --project-name=teaching
```

## Development workflow

1. Edit `.qmd`, `quarto render`, `./check-links.sh _site`.
2. Commit. **Push to `main` is the deploy** — CI renders, link-checks, then
   uploads to Cloudflare Pages. There is no separate release step.
3. Verify live. Cloudflare caches assets for 4 hours with `must-revalidate`; a
   `?cb=$RANDOM` query busts it when checking a change immediately.

## Known constraints

- **Publishing constraint (important).** Public files here must not name an
  institution, a module code, a term, or a scheduled course. Only courses
  actually in preparation are listed, without dates or links. If a change would
  add any of these, stop and ask first. This applies to `NEXT.md` and session
  notes too, which is why they are gitignored.
- **`_quarto.yml` renders only `*.qmd`.** A website project otherwise turns
  every loose `.md` in the project into a public page — `NEXT.md` became
  `_site/NEXT.html` before this was pinned down. Do not widen the `render:` list
  to include `.md` without checking what that would publish.
- **Cloudflare Pages project is named `teaching`**, not `teaching-hub`
  (`teaching.pages.dev` was globally taken, hence `teaching-62z.pages.dev`).
  `vars.CLOUDFLARE_PROJECT_NAME` must be `teaching` or the deploy job fails.
- **Custom domains cannot be scripted** — wrangler has no `pages domain`
  subcommand. Dashboard only, from the Pages project's Custom domains tab.
  Never hand-create the DNS record.
- **Inkscape SVGs need text converted to paths.** Inkscape's flowed text
  (`<flowRoot>`/`<flowPara>`) is SVG 1.2 and renders in **no** browser — labels
  vanish silently on the web while looking fine in Inkscape and in PDF exports.
  Export with `inkscape --export-text-to-path`. Inkscape 1.2.1's
  `text-convert-to-regular` action crashes headless.
- `styles/website.scss` is duplicated by hand with the template. A shared Quarto
  extension is deliberately deferred until several modules exist.

## Important architectural decisions

- **The hub is public and has no auth worker.** Everything on it is meant to be
  read by anyone. Access control belongs on course sites, not here.
- **The template stays public** — it is the OER anchor the hub points at. Making
  it private was proposed on 2026-07-21 and rejected for that reason.
- **Audience split drives content placement.** Rationale ("why HTML slides")
  lives here; operational instructions ("how to export a PDF", "use WSL2") live
  in the template's `guide.qmd`, behind the course password. When one repeats
  the other, the hub keeps the argument and the guide links to it.
- **Framing is employer-agnostic** — "open, reproducible teaching material",
  not "courses at institution X". This is deliberate positioning, not just
  caution: it survives changing employers without a rewrite.

## Things future sessions should always know

- Read `NEXT.md` and the newest file in `docs/sessions/` before starting.
- Both this repo and the template are **public**. Assume anything committed is
  world-readable, including commit messages and history. Force-pushing does not
  remove old commits from GitHub — they stay fetchable by full SHA.
- Template → module sync is by **cherry-pick**, never merge or rebase: module
  repos are created with "Use this template" and have unrelated histories. Keep
  infrastructure commits separate from content commits so they cherry-pick
  cleanly.
- `gh` is installed and authenticated as `hoelzer`; `wrangler` is authenticated
  via OAuth. Cloudflare account ID `6398bee0e2141168cd3fccf8cfbfe6ee`.

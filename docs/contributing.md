# Contributing to the Knowledge Base

This site is generated from the `docs/` folder in this repo with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/). Anyone in the
org can edit it via a normal PR, GitHub Actions rebuilds and redeploys on
merge to `main`.

## Adding a page

1. Create a `.md` file in the relevant folder (or a new folder if it's a new topic).
2. If it's not auto-discovered, add it to the `nav:` block in `mkdocs.yml`.
3. Open a PR. A preview isn't wired up yet. For local preview:
   ```
   pip install -r requirements.txt
   mkdocs serve
   ```

## Adding large files (slides, PDFs, VM images, recordings)

Git is a bad place for anything much over ~50-100MB. Rule of thumb:

- **Markdown, small images/diagrams, code, Terraform, PDFs under ~20MB** get committed
  directly to this repo.
- **PowerPoints, large PDFs, zipped tool releases (20MB-2GB)** get attached as a
  [GitHub Release](https://github.com/kuisc/knowledge-base/releases) asset on this
  repo, then link to it from the relevant doc page. Tag releases by category, e.g.
  `presentations-2026-fall`.
- **Large objects (>2GB)** should be stored on the Sharepoint by a club officer and
  linked to from the relevant doc page. Try and keep these to a minimum.

## Style

- One topic per page. Prefer several short pages over one giant page.
- Every folder should have an `index.md` that explains what's in it and links
  to its children.
- Date meeting notes and writeups (`YYYY-MM-DD-title.md`).

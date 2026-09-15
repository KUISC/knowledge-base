# Contributing to the Knowledge Base

This site is generated from the `docs/` folder in this repo with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/). Anyone in the
org can edit it via a normal PR — GitHub Actions rebuilds and redeploys on
merge to `main`.

## Adding a page

1. Create a `.md` file in the relevant folder (or a new folder if it's a new topic).
2. If it's not auto-discovered, add it to the `nav:` block in `mkdocs.yml`.
3. Open a PR. A preview isn't wired up yet — for local preview:
   ```
   pip install -r requirements.txt
   mkdocs serve
   ```

## Adding large files (slides, PDFs, VM images, recordings)

Git (even with LFS) is a poor place for anything much over ~50-100MB, and org
storage quotas aren't infinite. Rule of thumb:

- **Markdown, small images/diagrams, code, Terraform, PDFs under ~20MB** → commit
  directly to this repo.
- **PowerPoints, large PDFs, zipped tool releases (20MB-2GB)** → attach as a
  [GitHub Release](https://github.com/kuisc/knowledge-base/releases) asset on this
  repo, then link to it from the relevant doc page. Tag releases by category, e.g.
  `presentations-2026-fall`, `ctf-vmimages-2026`.
- **VM images / disk images (multi-GB)** → these don't belong in git at all.
  Host them externally (bucket, NAS, whatever the club settles on) and just
  document the download link + checksum + build instructions (ideally as
  Packer/Terraform so the image is reproducible and the binary itself is
  disposable).

## Style

- One topic per page. Prefer several short pages over one giant page.
- Every folder should have an `index.md` that explains what's in it and links
  to its children — don't rely on the nav sidebar alone.
- Date meeting notes and writeups (`YYYY-MM-DD-title.md`).

# KUISC Knowledge Base

Source for the club's knowledge base site, built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/)
and deployed to GitHub Pages on every push to `main`.

## Local development

```
pip install -r requirements.txt
mkdocs serve
```

Then visit http://127.0.0.1:8000.

## Structure

See [docs/index.md](docs/index.md) for the content map and
[docs/contributing.md](docs/contributing.md) for how to add pages and where
large files (slides, PDFs, VM images) should go.

## Repo layout

```
docs/                  # site content (markdown)
infra/terraform/       # actual Terraform for club-managed infra (not docs)
infra/packer/          # VM image build definitions
tools/                 # custom club tooling (source), documented under docs/tooling
mkdocs.yml             # site config/nav
.github/workflows/     # CI: builds and deploys the site via mkdocs gh-deploy
```

# docs

[![docs](https://github.com/go-sparse/docs/actions/workflows/docs.yml/badge.svg)](https://github.com/go-sparse/docs/actions/workflows/docs.yml)
[![License](https://img.shields.io/badge/license-BSD--3--Clause-A8372C?style=flat-square)](LICENSE)

**The go-sparse documentation site**, published at
<https://go-sparse.github.io/docs/>.

A [Hugo](https://gohugo.io/) site with the [tannevaled/hextra](https://github.com/tannevaled/hextra)
fork of the [Hextra](https://github.com/imfing/hextra) theme, built the same way
as the [go-numerics documentation](https://github.com/go-numerics/docs), with
go-sparse's branding. It documents
[`go-sparse/sparse`](https://github.com/go-sparse/sparse), released v0.1.0.

## Versions

`params.libraries` in `hugo.yaml` names the release of each library the pages
describe. A version of the documentation is **the newest minor version among
them**, or `main` while none is tagged:

| Source | Published under |
| --- | --- |
| branch `main`, with `params.libraries` in `hugo.yaml` | `https://go-sparse.github.io/docs/<MAJOR.MINOR>/`, or `/docs/main/` |
| the newest version (else `main`) | also `https://go-sparse.github.io/docs/latest/` |

`https://go-sparse.github.io/docs/` redirects to `latest/`. Each push to
`main` replaces the directory of the version it describes, and nothing else.
The pages describe **sparse v0.1.0**, published under `0.1/`.

## Layout

| Path | What |
| --- | --- |
| `content/en/` | the pages; `_index.md` is the home, one directory per library |
| `i18n/en.yaml` | the site's strings, beside the theme's |
| `hugo.yaml` | `baseURL`, the library versions described, the navbar, the brand mounts |
| `layouts/_partials/custom/` | the version selector and the outdated-version banner (read `versions.json`) |
| `layouts/_partials/favicons.html` | the brand's favicons |
| `assets/css/custom.css` | the brand red as Hextra's primary colour |
| `themes/hextra` | submodule: [tannevaled/hextra](https://github.com/tannevaled/hextra), pinned to v0.16.0 |
| `branding` | submodule: [go-sparse/brand](https://github.com/go-sparse/brand) |
| `scripts/publish-version.sh` | puts one build into a `gh-pages` checkout and rewrites `versions.json`, `latest` and the root redirect |

## Working locally

```sh
git clone --recurse-submodules https://github.com/go-sparse/docs.git
cd docs
mkdir -p data && sh themes/hextra/scripts/page-history.sh > data/pagehistory.json
hugo server --baseURL http://localhost:1313/docs/0.1/
```

then open <http://localhost:1313/docs/0.1/>. Hugo 0.146 or later.

## Publication

`.github/workflows/docs.yml`:

| Job | When | What |
| --- | --- | --- |
| `build` | pull requests, `main` | builds with `--baseURL …/docs/<version>/`, checks internal links and anchors (lychee, offline), uploads the site |
| `links` | pull requests, `main` | fetches every external URL the pages name |
| `deploy` | `main` | `scripts/publish-version.sh` into `gh-pages` (started empty on the first deploy), then a commit and a push |

GitHub Pages serves the `gh-pages` branch.

## Licence

BSD-3-Clause.

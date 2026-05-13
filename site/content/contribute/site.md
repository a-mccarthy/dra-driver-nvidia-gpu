---
title: Contribute to Docs
linkTitle: Docs site
weight: 15
description: Build and preview this site locally, and how it gets deployed.
---

The site you're reading lives in
[`site/`](https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu/tree/main/site)
and is built with [Hugo](https://gohugo.io/) and the
[Docsy](https://www.docsy.dev/) theme. This page covers what you need
to make a docs change locally and what happens when your PR merges.

## Prerequisites

| Tool | Minimum version | Notes |
| --- | --- | --- |
| [Hugo Extended](https://github.com/gohugoio/hugo/releases) | `0.125.0` | The **extended** build is required because Docsy compiles SCSS with Dart Sass. The non-extended build (including the one shipped by `apt install hugo`) will not work. |
| [Go](https://go.dev/dl/) | `1.25.0` | Hugo Modules uses `go` to fetch the Docsy theme declared in [`site/go.mod`](https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu/blob/main/site/go.mod). |
| [Node.js](https://nodejs.org/) + npm | Node 18+ LTS | Installs the PostCSS toolchain (`autoprefixer`, `postcss`, `postcss-cli`) that Hugo invokes during the build. |
| Git | any recent | Required by Hugo Modules and by `enableGitInfo = true` in [`site/hugo.toml`](https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu/blob/main/site/hugo.toml). |

The exact versions Netlify uses for the production build are pinned in
[`netlify.toml`](https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu/blob/main/netlify.toml)
at the repo root. Match them locally if you want byte-identical output.

## Local development

From inside `site/`:

```bash
npm ci          # one-time; installs the PostCSS deps locked in package-lock.json
npm run serve   # starts Hugo's live-reload dev server on http://localhost:1313
```

`npm run serve` runs `hugo server --bind 0.0.0.0 --buildDrafts --buildFuture`,
so pages with `draft: true` or a future `date:` in their front matter are
visible. Edit any file under `content/` and the browser reloads automatically.
The `--bind 0.0.0.0` flag exposes the server on all interfaces, which is
convenient under WSL or a remote dev container.

To produce a production build identical to what Netlify runs:

```bash
npm run build   # hugo --gc --minify; output written to ./public
```

## Where content lives

- `site/content/_index.md` — landing page
- `site/content/docs/` — the **Docs** top-nav section (concepts, guides,
  install/upgrade/uninstall, troubleshooting, reference)
- `site/content/contribute/` — the **Contribute** top-nav section (this page,
  development, proposals)
- `site/hugo.toml` — site configuration (top nav, params, markup, output formats)
- `site/go.mod` — pins the Docsy theme version
- `site/package.json` — npm scripts and PostCSS deps used at build time

Each page uses Docsy front matter, for example:

```markdown
---
title: My new page
linkTitle: My page
weight: 30
description: One-sentence summary that shows up in section listings.
---

Body content in Markdown.
```

The left sidebar orders pages by `weight:` ascending — pages without a
`weight` float to the top of their section.

## Deployment

The site is built and hosted by Netlify, configured by the repo-root
[`netlify.toml`](https://github.com/kubernetes-sigs/dra-driver-nvidia-gpu/blob/main/netlify.toml):

- **Production:** every push to `main` triggers a deploy of
  `hugo --gc --minify`, published to
  [dra-driver-nvidia-gpu.sigs.k8s.io](https://dra-driver-nvidia-gpu.sigs.k8s.io/).
- **Deploy previews:** every pull request gets its own preview URL built with
  `--buildDrafts --buildFuture`, so reviewers can see in-progress content.
  Netlify posts the preview URL back to the PR as a GitHub commit check.

DNS for the `sigs.k8s.io` subdomain is managed in the
[`kubernetes/k8s.io`](https://github.com/kubernetes/k8s.io) repository, not
in this repo.

## Common gotchas

- **Wrong Hugo build.** `apt install hugo` and the default Homebrew formula
  install the *non-extended* Hugo and will fail inside Docsy's SCSS partial.
  Install `hugo_extended` from the
  [GitHub release page](https://github.com/gohugoio/hugo/releases).
- **Skipping `npm ci`.** Without `node_modules/`, the build fails the moment
  Hugo's PostCSS pipeline runs (`postcss: command not found`). The error
  message does not make the connection to `package.json` obvious.
- **Drafts disappear in production.** A page with `draft: true` shows up
  under `npm run serve` but is stripped by `npm run build` and the Netlify
  production deploy. Drop `draft:` (or set it to `false`) when the page is
  ready to publish.
- **Hugo Modules need network access on the first build.** If you're behind
  a corporate proxy, set `HTTPS_PROXY` before running `hugo` so it can fetch
  the Docsy theme from GitHub. The `[module] proxy = "direct"` setting in
  `site/hugo.toml` deliberately bypasses the public Go module proxy.

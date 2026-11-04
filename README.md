# Riebeckite E2E: separate repositories — site

The site repository for the **separate repositories** Riebeckite sample. Content lives in the companion Vault repository.

- **Published site:** <https://repositories.riebeckite.dev>
- **Vault repository:** [Rerurate514/riebeckite-e2e-vault](https://github.com/Rerurate514/riebeckite-e2e-vault)
- **Riebeckite documentation:** <https://riebeckite.dev>
- **Other samples:** [integrated](https://integrated.riebeckite.dev) · [separate directories](https://directories.riebeckite.dev)

## Purpose

This repository is for teams that keep writing and site engineering in separate repositories with separate permissions and release cycles. The site repository is the only place that builds and deploys. The Vault repository is the only place that stores Markdown.

## Directory structure

```text
.
├── app/                       Riebeckite site code
├── public/                    Static files served as-is
├── content/                   Vault checkout (created locally and in CI)
├── riebeckite.config.ts       Site settings and plugins
├── wrangler.jsonc             Cloudflare Worker and custom domain
├── package.json
├── .github/workflows/
│   └── publish.yml            Build and deploy on push or repository_dispatch
└── github/
    └── notify-site.yml        Copy this file into the Vault repository
```

`content/` is intentionally not committed. The workflow checks out the Vault into that path, and local development uses your own Vault clone there.

## Requirements

- Node.js 22 or newer
- npm
- A Cloudflare account with a `riebeckite.dev` zone
- Access to the Vault repository

## Setup

```sh
git clone https://github.com/Rerurate514/riebeckite-e2e-site.git
cd riebeckite-e2e-site
git clone https://github.com/Rerurate514/riebeckite-e2e-vault.git content
npm ci
```

`riebeckite.config.ts` keeps `content.directory: "content"`, so no path change is needed.

## Local development

```sh
npm exec riebeckite check
npm exec riebeckite dev
```

Edit files in `content/` and push them to the Vault repository when they are ready. Do not commit the nested Vault checkout to this repository.

Useful commands:

```sh
npm exec riebeckite doctor
npm run build
npm exec riebeckite inspect
```

## Build

```sh
npm run build
```

The output is `dist/`. The build reads the Vault checkout in `content/`.

## Deploy

The Worker is named `riebeckite-e2e-site` and is attached to `repositories.riebeckite.dev` by `wrangler.jsonc`.

```sh
npx wrangler login
npx wrangler deploy
```

The first deploy creates the Worker and the custom domain.

## GitHub Actions

### Site repository: `publish.yml`

Runs on pushes to `main`, manual dispatch, and `repository_dispatch` events of type `content-updated`. It checks out this repository, checks out the Vault into `content/`, builds, validates, deploys, and records publication state.

### Vault repository: `notify-site.yml`

Copy `github/notify-site.yml` to `.github/workflows/notify-site.yml` in the Vault repository. It sends a `content-updated` dispatch to this repository on every push to the Vault's `main`.

Required secrets:

| Repository | Secret | Purpose |
| --- | --- | --- |
| site | `CLOUDFLARE_API_TOKEN` | Workers Scripts:Edit for the account |
| site | `CLOUDFLARE_ACCOUNT_ID` | Cloudflare account ID |
| site | `RIEBECKITE_CONTENT_READ_TOKEN` | Optional. Only needed if the Vault is private. Public Vaults are read with the workflow's `github.token`. |
| vault | `SITE_DISPATCH_TOKEN` | Fine-grained token scoped to this repository with **Contents: read and write**, used to create the dispatch. |

## Content

All content is in the Vault repository. This repository only contains code, configuration, and the static attachment in `public/assets/sample.txt`.

## Related documentation

- [Quick Start](https://riebeckite.dev/getting-started/quick-start)
- [Separate content repository concepts](https://riebeckite.dev/guides/deployment/separate-content-repository/concepts)
- [Repository split](https://riebeckite.dev/guides/deployment/separate-content-repository/repository-split)
- [Publishing](https://riebeckite.dev/guides/deployment/separate-content-repository/publishing)
- [Cloudflare Workers deployment](https://riebeckite.dev/guides/deployment/cloudflare-workers)

## License

Apache License 2.0. See [LICENSE](LICENSE).

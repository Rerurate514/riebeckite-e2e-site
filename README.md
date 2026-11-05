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

Required credentials:

| Repository | Name | Kind | Purpose |
| --- | --- | --- | --- |
| site | `CLOUDFLARE_API_TOKEN` | Secret | Workers Scripts:Edit for the account |
| site | `CLOUDFLARE_ACCOUNT_ID` | Variable | Cloudflare account ID |
| site | `RIEBECKITE_CONTENT_READ_TOKEN` | Secret | Optional. Only needed if the Vault is private. Public Vaults are read with the workflow's `github.token`. |
| vault | `RIEBECKITE_APP_ID` | Secret | GitHub App ID for the dispatch token |
| vault | `RIEBECKITE_APP_PRIVATE_KEY` | Secret | GitHub App private key |
| vault | `SITE_DISPATCH_TOKEN` | Secret | Optional fallback instead of the GitHub App |

The GitHub App needs **Contents: read and write** and **Metadata: read-only**, and must be installed only on this site repository. The workflow mints a short-lived installation token for each dispatch.

## Content

All content is in the Vault repository. This repository only contains code, configuration, and the static attachment in `public/assets/sample.txt`.

## Continuous E2E

This repository tracks the latest stable `@riebeckite/*` release, so it tests the package as an external consumer rather than through a workspace link.

To validate a release candidate, create a branch, pin the candidate version in `package.json`, run `npm install`, `npm exec -- riebeckite check`, and `npm exec -- riebeckite build`, then push the branch and run the publish workflow with `workflow_dispatch`. Keep the version in the commit message. Do not merge the candidate branch until the deploy succeeds; `main` remains on the stable release.

A candidate that changes the content checkout contract also needs a run from the Vault repository to verify the dispatch path.

## Related documentation

- [Quick Start](https://riebeckite.dev/getting-started/quick-start)
- [Project structure](https://riebeckite.dev/getting-started/project-structure)
- [Separate repositories setup](https://riebeckite.dev/getting-started/separate-repositories-setup)
- [Cloudflare Workers deployment](https://riebeckite.dev/guides/deployment/cloudflare-workers)
- [GitHub Actions deployment](https://riebeckite.dev/guides/deployment/github-actions)

## License

Apache License 2.0. See [LICENSE](LICENSE).

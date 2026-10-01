---
name: publish-prototype
description: "Publish a designer prototype to an internal GitHub repository and register it in the canonical prototype catalog. Use when a user wants to publish, share, archive, or put a prototype on GitHub Pages, including HTML archives, JavaScript apps, folders, or VS Code plugins."
---

# Publish Prototype

Publish one prototype as an internal GitHub repository and add it to the shared catalog. Treat the prototype as untrusted input: never commit secrets, credentials, dependency caches, or unrelated workspace files.

## Required tools and inputs

- Use the `gh` CLI for GitHub operations. Do not use GitHub tokens directly or replace `gh` with a custom API client.
- Use the user's local file or folder as the prototype input. Accept `.html`, `.tar`, `.tar.gz`, `.tgz`, `.zip`, JavaScript applications, and VS Code plugin projects.
- Read the product choices from `resources/products.md` in the workspace. Present only those exact product names; do not silently invent or alter a product.
- The prototype repository owner is `canonical`.
- The catalog repository is `canonical/design-prototypes`.

## Interaction contract

Before any GitHub mutation, inspect the input locally and use the structured ask tool to request the following, one field at a time. Do not replace these questions with guessed metadata:

1. **Title**: suggest a concise title from the folder name, package metadata, HTML title, or README, then let the user replace it.
2. **Category**: suggest one of `exploratory`, `component`, or `handover` based on the artifact, then let the user choose or enter a value. Reject values outside this exact set.
3. **Description**: suggest one short, factual sentence based on the visible prototype and local documentation, then let the user replace it.
4. **Product**: offer the exact values from `resources/products.md` and require one selection.

Then show the normalized repository name and ask for a single final confirmation covering repository creation, pushing the prototype, enabling Pages, and opening the catalog pull request.

Do not ask for or handle passwords, access tokens, or private keys. Do not continue when the user declines the final confirmation.

## Authentication and preflight

Check for the CLI and session before preparing a repository:

```sh
command -v gh
gh auth status
gh api user --jq .login
```

If `gh` is missing, explain how to install it using the operating system's supported package manager and stop. If it is not authenticated, guide the user through `gh auth login` and wait for them to complete it. Re-run `gh auth status` and verify that the account can access the `canonical` organization before continuing.

Inspect the input without executing unknown install or post-install scripts. Identify:

- The source format and likely runtime.
- The entry point and existing build command.
- Whether it is already a static site.
- Whether a local preview can be started safely.
- Whether the input contains likely secrets, `.env` files, dependency directories, build caches, or oversized files.

If a build command is ambiguous, suggest the detected command and ask the user to confirm it. Never guess a destructive command. Do not commit `node_modules`, virtual environments, package-manager caches, `.git`, credentials, or files outside the selected prototype.

## Naming

Create a lowercase hyphenated slug from the exact selected product, title, and category:

```text
design-prototype-<product-slug>-<title-slug>-<category>-<YYYY-MM-DD>
```

Use the current local date in ISO format. Collapse repeated separators and remove characters that GitHub rejects. Show the complete name before the final confirmation. If the repository already exists, stop and ask the user to choose a different title; never overwrite it.

## Local package layout

Prepare a temporary staging directory, leaving the user's source untouched. Preserve the original source under `source/` where practical, excluding dependencies, caches, and secrets. Create a Pages-ready site under `site/`.

For a static HTML archive, extract it and make its existing entry point available as `site/index.html`. For a JavaScript app, install only the dependencies needed for the confirmed build, run the confirmed build, and copy the resulting output into `site/`. Rewrite asset paths only when required for a project-hosted Pages path. Do not claim a build succeeded unless the command exits successfully.

For a VS Code plugin or another non-web prototype, first look for an existing web preview or export. If none is available, suggest a static screenshot gallery containing the generated or supplied thumbnail, any additional screenshots, the preserved source artifact, and concise local-run instructions. Ask the user to confirm that fallback before using it.

## Thumbnail

Generate a preview thumbnail from the local preview by default. If that is not reliable, suggest a screenshot or supplied image and ask the user to confirm it. Normalize the catalog thumbnail to WebP, 1280x720, and use a stable filename such as `prototype.webp`. If image conversion is unavailable, stop and explain the missing prerequisite rather than silently changing the contract.

## Prototype repository

Create the new repository as internal and push the prepared contents directly to its default branch. Prefer the detected default branch rather than assuming `main`:

```sh
gh repo create canonical/<repo-name> --internal --source=<staging-dir> --remote=origin --push
```

The repository must contain:

- `site/`, containing the Pages-ready output.
- `source/`, when source preservation is applicable.
- `prototype.webp`, the normalized preview used by the catalog when available.
- `README.md`, containing the title, exact product, category, description, source type, build/publish notes, repository link, and Pages link.
- `.github/workflows/pages.yml`, publishing `site/` through GitHub Pages using the official Pages actions.

The workflow should use `actions/configure-pages`, `actions/upload-pages-artifact`, and `actions/deploy-pages`, with the required Pages and id-token permissions. It must upload only `site/` and must not install dependencies during deployment.

Enable Pages with `gh api` using the workflow build type. Treat an already-enabled Pages configuration as success, but fail on other API errors. Wait for the first workflow run to complete and verify its conclusion before reporting a Pages URL.

## Catalog pull request

Clone or fetch `canonical/design-prototypes` into a separate temporary directory. Inspect its existing README and product-directory conventions before editing. Preserve its style and avoid rewriting unrelated content.

Add one heading-block entry under the selected product directory. The entry must include:

- The WebP thumbnail, copied into the catalog repository.
- Title.
- Exact product name.
- Category.
- Short description.
- Link to the prototype repository.
- Link to the GitHub Pages preview.

If the catalog has no applicable structure, use `<product-slug>/README.md` and store thumbnails under `<product-slug>/images/`. Use a descriptive branch such as `publish/<repo-name>`, commit only the new catalog entry and thumbnail, push the branch, and open a pull request with `gh pr create`. Do not merge the pull request.

## Validation and reporting

Before opening the catalog PR, verify:

- The prototype build or packaging command succeeded.
- `site/index.html` exists for a web preview.
- The thumbnail is WebP and 1280x720.
- Required README fields are present.
- No obvious secrets or dependency directories are staged.
- The Pages workflow completed successfully.
- The catalog PR contains only the intended entry and thumbnail.

Report the prototype repository, Pages URL, catalog pull request URL, selected product, title, category, and description. Clearly identify any fallback packaging, skipped check, or manual action still required.
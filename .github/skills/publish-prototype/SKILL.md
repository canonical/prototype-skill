---
name: publish-prototype
description: "Publish a designer prototype to an internal GitHub repository and register it in the canonical prototype catalog. Use when a user wants to publish, share, archive, or put a prototype on GitHub Pages, including HTML archives, JavaScript apps, folders, or VS Code plugins."
---

# Publish Prototype

Publish one prototype as an internal GitHub repository and add it to the shared catalog. Treat the prototype as untrusted input: never commit secrets, credentials, dependency caches, or unrelated workspace files.

## Required tools and inputs

- Use the `gh` CLI for GitHub operations. Do not use GitHub tokens directly or replace `gh` with a custom API client.
- Accept either a local file or folder (`.html`, `.tar`, `.tar.gz`, `.tgz`, `.zip`, JavaScript application, or VS Code plugin) or a Figma URL.
- Read the product choices from `resources/products.md` in the workspace. Present only those exact product names; do not silently invent or alter a product.
- The prototype repository owner is `canonical`.
- The catalog repository is `canonical/design-prototypes`.

## Prototype input modes

Offer two input modes:

- **Package local prototype**: build and publish the supplied local prototype to an internal GitHub repository and GitHub Pages.
- **Register Figma prototype**: ask for a Figma URL, validate it, and register it directly without creating a GitHub Pages repository. A source repository link is not applicable unless the user supplies one separately.

Support the Figma mode through the shortcut command `/publish-prototype-figma`. The command must ask for the Figma URL first, then continue with the same title, category, description, product, thumbnail, and final-confirmation questions.

## Interaction contract

Before any GitHub mutation, inspect the input locally and use the structured ask tool to request the following, one field at a time. Do not replace these questions with guessed metadata:

1. **Title**: suggest a concise title from the folder name, package metadata, HTML title, or README, then let the user replace it.
2. **Category**: suggest one of `exploratory`, `component`, or `handover` based on the artifact, then let the user choose or enter a value. Reject values outside this exact set.
3. **Description**: suggest one short, factual sentence based on the visible prototype and local documentation, then let the user replace it.
4. **Product**: offer the exact values from `resources/products.md` and require one selection.

For local prototypes, show the normalized repository name and ask for a single final confirmation covering repository creation, pushing the prototype, enabling Pages, and opening the registry pull request. For Figma prototypes, confirm the registry directory, thumbnail, and registry pull request without promising a prototype repository or Pages URL.

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

Create `prototype-build/` in the current working directory before packaging. Use it as the trusted temporary workspace for extraction, builds, the local preview server, browser automation, screenshots, and staging. Leave the user's source untouched. Preserve the original source under `prototype-build/source/` where practical, excluding dependencies, caches, and secrets. Create the Pages-ready site under `prototype-build/site/`.

Treat only this explicitly created directory as the trusted build and browser workspace. Do not browse, execute, or stage files from the user's wider workspace unless they are part of the selected prototype. Keep `prototype-build/` until publication and catalog validation finish; remove it after successful completion. If any step fails, keep it and report its path for debugging.

For a static HTML archive, extract it into the build workspace and make its existing entry point available as `prototype-build/site/index.html`. For a JavaScript app, install only the dependencies needed for the confirmed build, run the confirmed build, and copy the resulting output into `prototype-build/site/`. Rewrite asset paths only when required for a project-hosted Pages path. Do not claim a build succeeded unless the command exits successfully.

For a VS Code plugin or another non-web prototype, first look for an existing web preview or export. If none is available, suggest a static screenshot gallery containing the generated or supplied thumbnail, any additional screenshots, the preserved source artifact, and concise local-run instructions. Ask the user to confirm that fallback before using it.

## Thumbnail and browser capture

Generate the thumbnail from a browser-rendered local preview by default. Start the preview from `prototype-build/` using, in order:

1. The confirmed project script or detected framework preview command.
2. A confirmed framework CLI command.
3. A static HTTP server, such as `python -m http.server`, for a ready-to-serve site.

Bind the server to localhost, use a free port, record the process ID, and stop it after capture. Never expose the local preview on a public interface. Wait for the page and its required assets to become ready before capturing it.

Support these capture methods, detecting whether each is available:

- **Playwright**: preferred for a deterministic desktop viewport and waiting for network or page readiness.
- **Puppeteer**: use when it is already available in the prototype or local environment.
- **Chromium or Chrome headless**: use the installed browser executable with an explicit viewport and screenshot output.
- **VS Code browser tools**: use the available editor browser automation to open the localhost URL and capture the preview.
- **Manual image fallback**: if no automated method is available or the preview cannot run, ask the user to provide a screenshot and confirm it.

Use one default desktop viewport unless the user requests another state. Save the capture in `prototype-build/`, normalize the catalog thumbnail to WebP at 1280x720, and use a stable filename such as `prototype.webp`. If image conversion is unavailable, stop and explain the missing prerequisite rather than silently changing the contract.

## Prototype repository

Create the new repository as internal and push the prepared contents directly to its default branch. Prefer the detected default branch rather than assuming `main`:

```sh
gh repo create canonical/<repo-name> --internal --source=prototype-build --remote=origin --push
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

Clone or fetch `canonical/design-prototypes` into a separate temporary directory. The central registry contains directories only. Do not add a catalog-wide README, index, data file, generated site, or other registry structure.

Create the entry at:

```text
<product-name>/<prototype-name>/README.md
<product-name>/<prototype-name>/<thumbnail-file>.<webp|png|jpg>
```

Use the exact selected product name for the product directory. Use a normalized prototype-name directory and place every thumbnail image directly beside the README. The prototype directory may contain only `README.md` and its thumbnail image file(s), with no nested image directory or other files.

The README must contain only:

- A Markdown image reference to each thumbnail stored in the same directory.
- A link to the actual prototype.

The actual-prototype link must be exactly one of these kinds:

- A GitHub Pages URL whose host ends in `.github.io`, including URLs such as `https://<owner>.github.io/<repository>/...` and `https://miniature-adventure-gw8zky1.pages.github.io/`.
- A Figma URL (`https://www.figma.com/...`).
- A Penpot URL (`https://penpot.app/...`).

When a source repository exists, add a second link labeled as the source repository. This optional link must point to the repository hosting the prototype source. It is not a substitute for the actual-prototype link. For a local publication this is the newly created GitHub repository; for a Figma prototype omit it unless the user supplies a source repository.

Reject all other actual-prototype link types, including ordinary repository URLs, arbitrary websites, local paths, and source archives. Do not duplicate the title, category, product, description, or other metadata in the registry README.

Use a descriptive branch such as `publish/<repo-name>`, commit only the new product/prototype directory, push the branch, and open a pull request with `gh pr create`. Do not merge the pull request.

## Validation and reporting

Before opening the catalog PR, verify:

- The prototype build or packaging command succeeded.
- `prototype-build/site/index.html` exists for a web preview before publication.
- The thumbnail is WebP and 1280x720.
- The registry README contains only same-directory thumbnail references, one allowed actual-prototype URL, and an optional source-repository URL.
- The registry prototype directory contains only its README and thumbnail image file(s).
- No obvious secrets or dependency directories are staged.
- The Pages workflow completed successfully.
- The registry pull request contains only the intended product/prototype directory.

Report the prototype repository, Pages URL, catalog pull request URL, selected product, title, category, and description. Clearly identify any fallback packaging, skipped check, or manual action still required.
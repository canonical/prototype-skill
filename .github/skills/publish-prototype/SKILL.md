---
name: publish-prototype
description: "Publish or register a design prototype in canonical/ai-design-ops. Use when a user wants to publish, share, archive, or put a prototype on GitHub Pages, register a Figma prototype, or document a prototype with screenshots when live publishing fails."
---

# Publish Prototype

Contribute one prototype to `Prototypes/` in `canonical/ai-design-ops`, using the Markdown metadata approach in `Projects/`. Register an existing live prototype, or attempt to publish a local prototype and document it with screenshots if live publishing fails. Treat the prototype as untrusted input: never commit secrets, credentials, dependency caches, or unrelated workspace files.

## Required tools and inputs

- Use the `gh` CLI for GitHub operations. Do not use GitHub tokens directly or replace `gh` with a custom API client.
- Accept a local file or folder (`.html`, `.tar`, `.tar.gz`, `.tgz`, `.zip`, JavaScript or Python application, or VS Code plugin), a Figma URL, or an existing GitHub Pages URL with its source repository URL.
- Read the product choices from this skill's `resources/products.md`. Present only those exact product names; do not silently invent or alter a product.
- The prototype repository owner is `canonical`.
- The catalog repository is `canonical/ai-design-ops`.
- Inspect the catalog's contribution guidance, `TAGS.md`, `Projects/_template.md`, a representative Projects entry, and any existing Prototypes template or validation before preparing an entry. Reuse the Projects metadata-line syntax and established labels rather than inventing a YAML schema. Do not hand-edit generated manifests.

## Prototype input modes

Offer three input modes leading to two upload paths:

- **Package local prototype**: attempt to build and publish the supplied local prototype to an internal GitHub repository and GitHub Pages. Successful publication uses the live-prototype path; a failed build or Pages deployment uses the screenshot-documentation path when actual screens can be captured.
- **Register Figma prototype**: ask for a Figma URL, validate it, and register it directly without creating a GitHub Pages repository. A source repository link is not applicable unless the user supplies one separately.
- **Register existing GitHub Pages prototype**: ask for the live GitHub Pages URL and its source repository URL, validate both, and register them without rebuilding or creating a repository.

Support the Figma mode through the shortcut command `/publish-prototype-figma`. Ask for the Figma URL first, then continue with the title, category, card summary, product, metadata, thumbnail, and final-confirmation questions. Do not ask for a long description.

### Upload path 1: live prototype

For a prototype hosted on Figma or GitHub Pages, contribute metadata, the selected one-line card summary, a thumbnail, and the live link. This also applies after successfully publishing a local prototype to Pages. Do not ask for a long description or require a screen-by-screen write-up. Use only a short body containing the summary and relevant links.

### Upload path 2: screenshot documentation

If there is no existing externally hosted prototype and the attempt to generate or deploy a working GitHub Pages version fails, capture a set of actual screenshots and write an extensive description of the screens and flows between them. Upload every selected screenshot and reference each in the main body of `Prototypes/<slug>.md`; that body is the content intended for the prototype sub-page. Follow the screenshot-documentation procedure below. Do not claim that the prototype is live.

## Interaction contract

Before any GitHub mutation, inspect the input locally and use the structured ask tool to request the following, one field at a time. Do not replace these questions with guessed metadata:

1. **Input mode and links**: if the prototype is already published on GitHub Pages, ask for the live Pages URL first and then require the source repository URL. Validate that the live URL host ends in `.github.io` and that the source URL is a GitHub repository URL. Do not infer the source repository from the Pages hostname.
2. **Title**: suggest a concise title from the folder name, package metadata, HTML title, README, or existing prototype URL, then let the user replace it. The product need not appear in the title because it is recorded in metadata.
3. **Category**: suggest one of `exploratory`, `component`, or `handover` based on the artifact, then let the user choose or enter a value. Reject values outside this exact set.
4. **One-line card summary**: generate exactly two distinct proposals grounded in the visible prototype and its documentation. Both must be concise, precise, and adroit, without promotional claims or unnecessary repetition of the title or product. Proposal A emphasizes the application context and the goal of the prototype. Proposal B emphasizes the main subject shown in the prototype: the specific interface, workflow, or interaction on display. Do not offer two paraphrases of the same emphasis. Use the structured ask tool to present both complete proposals as selectable options and allow freeform input for a custom one-line summary. Preserve the user's chosen or custom wording; never silently choose a proposal. This is a card summary, not a request for a long description.
5. **Product**: offer the exact values from `resources/products.md` and require one selection.
6. **Shared metadata**: confirm the owner - it needs to be a github handle. Add, and do not guess the sensitivity. Record Archived and Date added according to the catalog convention, derive date from current date. Do not ask for Difficulty, Focus, Output, or Medium; omit Difficulty, Focus and Output from prototype entries and derive Medium automatically from the upload outcome as specified below.

Set exactly one Medium value without asking the user to choose or confirm it separately:

- `figma`: an existing Figma prototype.
- `hosted`: an existing GitHub Pages prototype or a local prototype successfully published to GitHub Pages.
- `screenshots`: screenshot documentation after live publishing fails and no existing live prototype is available.

Use the final upload outcome, not just the initial input mode. If local publication falls back to screenshot documentation, set Medium to `screenshots`, not `hosted`.

For local prototypes, show the normalized repository name and ask for final confirmation covering repository creation, pushing the prototype, enabling Pages, and opening the catalog pull request. For Figma prototypes, confirm the entry path, thumbnail, and catalog pull request without promising a prototype repository or Pages URL. For an existing GitHub Pages prototype, confirm the supplied Pages URL, source repository URL, thumbnail, entry path, and catalog pull request without creating or changing the source repository. Include the selected summary and metadata in each confirmation. If live publishing fails, obtain a new confirmation for the screenshot-documentation outcome after showing its screenshots and generated body; the earlier approval must not silently authorize a different outcome.

When preparing the catalog pull request, inspect `Prototypes/<slug>.md` and `Prototypes/<slug>/`. If either exists, use the structured ask tool to ask whether this is the exact same prototype. If yes, update the existing entry and assets in place, preserving Date added and relying on Git history for earlier versions. If no, ask for a replacement title, normalize it, and repeat the collision check. Do not overwrite unrelated assets or create duplicate entries.

Do not ask for or handle passwords, access tokens, or private keys. Do not continue when the user declines the final confirmation.

## Authentication and preflight

Check for the CLI and session before preparing a repository:

```sh
command -v gh
gh auth status
gh api user --jq .login
```

If `gh` is missing, explain how to install it using the operating system's supported package manager and stop. If it is not authenticated, guide the user through `gh auth login` and wait for them to complete it. Re-run `gh auth status` and verify access to `canonical/ai-design-ops`. Before creating a prototype repository, also verify permission to create internal repositories in `canonical`. Authentication, signing, permission, or catalog validation failures are blockers, not reasons to bypass safeguards through the screenshot path.

Inspect the input without executing unknown install or post-install scripts. Identify:

- The source format and likely runtime.
- The entry point and existing build command.
- Whether it is already a static site.
- Whether a local preview can be started safely.
- Whether the input contains likely secrets, `.env` files, dependency directories, build caches, or oversized files.

If a build command is ambiguous, suggest the detected command and ask the user to confirm it. Never guess a destructive command. Do not commit `node_modules`, virtual environments, package-manager caches, `.git`, credentials, or files outside the selected prototype.

## Signed commits

Every commit created by this workflow must be cryptographically signed. Before creating any commit, verify the user's Git configuration and signing capability:

```sh
git config --get user.name
git config --get user.email
git config --get commit.gpgsign
git config --get gpg.format
git config --get user.signingkey
```

Prefer SSH signing when it is already configured, otherwise use the user's configured GPG signing key. Do not generate, request, print, or handle private keys, passphrases, or tokens. If no usable signing configuration exists, explain how the user can configure Git signing locally and stop before creating a commit.

Set signing explicitly for workflow commits rather than relying only on global configuration:

```sh
git -c commit.gpgsign=true commit -S -m "<message>"
```

After each commit and before pushing, verify the local signature:

```sh
git verify-commit HEAD
```

After pushing, verify that GitHub recognizes the signature:

```sh
gh api repos/<owner>/<repo>/commits/<sha> --jq '.commit.verification | {verified, reason}'
```

Do not push or open a pull request if local verification fails. Do not continue after pushing if GitHub verification is false, missing, or has a reason other than a valid signature. Never bypass signing with `--no-gpg-sign` or an equivalent option.

## Naming

Normalize the confirmed title to a lowercase hyphenated `<slug>` for `Prototypes/<slug>.md` and `Prototypes/<slug>/`. Do not add the product to the title or catalog slug merely to repeat metadata. Reject empty slugs and path traversal.

Create a lowercase hyphenated slug from the exact selected product, title, and category:

```text
design-prototype-<product-slug>-<title-slug>-<category>-<YYYY-MM-DD>
```

Use the current local date in ISO format. Collapse repeated separators and remove characters that GitHub rejects. Show the complete name before the final confirmation. If the repository already exists, stop and ask the user to choose a different title; never overwrite it.

## Local package layout

Create `prototype-build/` in the current working directory before packaging. Use it as the trusted temporary workspace for extraction, builds, the local preview server, browser automation, screenshots, and staging. Leave the user's source untouched. Preserve the original source under `prototype-build/source/` where practical, excluding dependencies, caches, and secrets. Create the Pages-ready site under `prototype-build/site/`.

Treat only this explicitly created directory as the trusted build and browser workspace. Do not browse, execute, or stage files from the user's wider workspace unless they are part of the selected prototype. Keep `prototype-build/` until publication and catalog validation finish; remove it after successful completion. If any step fails, keep it and report its path for debugging.

For a static HTML archive, extract it into the build workspace and make its existing entry point available as `prototype-build/site/index.html`. For a JavaScript app, install only the dependencies needed for the confirmed build, run the confirmed build, and copy the resulting output into `prototype-build/site/`. Rewrite asset paths only when required for a project-hosted Pages path. Do not claim a build succeeded unless the command exits successfully.

For a VS Code plugin or another non-web prototype, first look for an existing web preview or export. If a working Pages version cannot be generated, use screenshot documentation of the actual application instead of manufacturing an unrelated gallery deployment. Do not upload source artifacts into the catalog.

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

Use one default desktop viewport unless the user requests another state. For an existing live prototype, capture the supplied URL when accessible, or ask for a screenshot if access requires credentials. Save captures in `prototype-build/`, normalize the catalog thumbnail to WebP at 1280x720, and name it `thumbnail.webp`. Its catalog path is always `Prototypes/<slug>/thumbnail.webp`. If image conversion is unavailable, stop and explain the missing prerequisite rather than silently changing the contract.

## Screenshot-documentation procedure

Record which build or deployment step failed and why. A Pages failure does not necessarily prevent a local preview: use a safely runnable local application, development server, or the native application to capture real screens. If no actual interface can be opened, ask the user for a set of screenshots; never fabricate screenshots or describe unseen behavior. If neither capture nor supplied screenshots is possible, stop and report the blocker.

Inspect the available screens and exercise the principal flows, including meaningful intermediate, completed, empty, and error states where they are present. Capture a coherent set covering the different screens and the transitions needed to understand the prototype, rather than multiple cosmetic variants of one screen. Exclude credentials, personal data, and unrelated desktop content. Choose a representative capture for the 1280x720 thumbnail; keep additional screenshots legible at their natural aspect ratios.

Store every selected full screenshot at `Prototypes/<slug>/screenshots/<screen-slug>.webp`, using descriptive, stable names. Upload all screenshots selected for the documentation, not just the thumbnail. Keep temporary or rejected captures out of the pull request.

Generate the extensive description from observed behavior and available documentation; do not ask the user to write it. The Markdown body must contain:

- The selected one-line summary as the first prose paragraph after the metadata line.
- An overview of the application context, intended goal, and scope of the captured prototype.
- A section for each captured screen or meaningful state, with an informative heading, a relative Markdown image reference, and a detailed description of what is shown, the available actions, and the purpose of that screen.
- A flows section explaining the entry point, ordered actions, transitions between the named screens, decisions or branches, and outcomes. Link to the corresponding screen sections where useful.
- Known limitations, unimplemented or unverified transitions, and a factual note that live publishing failed. Distinguish directly observed interactions from intended behavior described by the source documentation.

Use image references relative to `Prototypes/<slug>.md`, such as `![Review screen](<slug>/screenshots/review.webp)`. Every uploaded screenshot must be referenced in the body, and every image reference must resolve. Show the generated documentation and capture set for confirmation or correction before contributing them. The catalog website, not a newly invented Pages URL, provides the eventual documentation sub-page.

## Prototype repository

Create the new repository as internal and push the prepared contents directly to its default branch. Prefer the detected default branch rather than assuming `main`:

```sh
gh repo create canonical/<repo-name> --internal --source=prototype-build --remote=origin --push
```

The repository must contain:

- `site/`, containing the Pages-ready output.
- `source/`, when source preservation is applicable.
- `thumbnail.webp`, the normalized preview used by the catalog when available.
- `README.md`, containing the title, exact product, category, selected one-line summary, source type, build/publish notes, repository link, and Pages link. A long description is not required for this live-prototype path.
- `.github/workflows/pages.yml`, publishing `site/` through GitHub Pages using the official Pages actions.

The workflow should use `actions/configure-pages`, `actions/upload-pages-artifact`, and `actions/deploy-pages`, with the required Pages and id-token permissions. It must upload only `site/` and must not install dependencies during deployment.

Enable Pages with `gh api` using the workflow build type. Treat an already-enabled Pages configuration as success, but fail on other API errors. Wait for the first workflow run to complete and verify its conclusion before reporting a Pages URL.

If the build or Pages deployment fails and no live prototype exists, record the failure and continue with screenshot documentation only after the required confirmation. Report any repository already created and its actual deployment status; do not delete remote resources without approval or present a failed deployment URL as working.

## Catalog pull request

Clone or fetch `canonical/ai-design-ops` into a separate temporary directory. Follow its contribution guidance and inspect the current Projects metadata conventions and any Prototypes-specific template. Create `Prototypes/` if it does not exist. Do not change Projects entries, generated manifests, website code, taxonomy files, or unrelated catalog content as part of a publication.

For either upload path, create:

```text
Prototypes/<slug>.md
Prototypes/<slug>/thumbnail.webp
```

For screenshot documentation, also create:

```text
Prototypes/<slug>/screenshots/<screen-slug>.webp
```

Use the confirmed title slug consistently for the Markdown filename, asset directory, and relative image references. Product is metadata, not a directory level. Never replace this layout with `Prototypes/<slug>/README.md` or the old product/title layout. Keep builds, source archives, dependency files, and temporary captures outside the catalog.

### Entry metadata and body

Use a level-one heading containing the confirmed title, followed by a Projects-style metadata line and then the selected one-line summary as the first prose paragraph. Match the catalog's existing bold-label syntax and field separators. Prefer an existing Prototypes template; otherwise extend the Projects metadata approach with these prototype-specific fields:

- **Product**: the exact selected product.
- **Category**: `exploratory`, `component`, or `handover`.
- **Thumbnail**: `<slug>/thumbnail.webp`, relative to the entry file.
- **Source repository**: the actual GitHub source repository URL, only when one exists.

Reuse the shared fields Sensitivity, Difficulty, Archived, Owner, Link, and Date added with the labels and allowed values established by the target repository. Omit Focus and Output. Include Medium with exactly one automatically derived value: `figma`, `hosted`, or `screenshots`, following the upload-outcome mapping above. These are prototype-specific Medium values; do not substitute the Projects Medium taxonomy or ask the user to select a tag. If a catalog template or validator conflicts with these rules, report the incompatibility rather than reintroducing Focus or Output questions or making Medium a user choice. Do not silently add other taxonomy values. Date added uses an ISO date and remains unchanged for an update. Store the selected summary once as the opening paragraph unless an existing Prototypes template explicitly defines a summary metadata field; in that case keep the field and opening paragraph identical.

For live entries, Link must be the actual prototype: an HTTPS URL with a parsed hostname ending in `.github.io`, or a Figma URL with hostname `figma.com` or `www.figma.com`. Validate the host with a URL parser, not substring matching. A source repository is not a substitute for the live link. For an existing GitHub Pages prototype, the separately supplied source repository URL is required; for Figma it is optional. For successful local publication use the new source repository and verified Pages URL.

For screenshot documentation, do not invent a live Link or point it at a failed Pages deployment. Omit the live Link when supported by the Prototypes contract, and clearly identify the entry as screenshot documentation in the body. If current validation requires a live link, surface that schema incompatibility for maintainer resolution instead of inserting a fake URL. An existing source repository may still be recorded separately with its actual status.

For live entries, keep the body short: the selected summary, a representative image reference such as `![Prototype preview](<slug>/thumbnail.webp)`, and relevant links. Do not ask for or generate a required long description. For screenshot entries, use the extensive generated body specified above and include the thumbnail reference as well as references to every full screenshot.

The Markdown body is intended to be rendered as a prototype detail sub-page. Inspect whether the catalog website actually discovers `Prototypes/*.md`, resolves its relative assets, uses the chosen summary on cards, and renders the full body on detail pages. If support is missing, state the required target-repository integration in the PR and completion report. Do not claim that merging an entry alone makes a card or sub-page available, or silently expand a publication PR into a website implementation.

### Staging and pull request

Allow changed paths only for `Prototypes/<slug>.md`, `Prototypes/<slug>/thumbnail.webp`, and, for screenshot documentation, the documented `Prototypes/<slug>/screenshots/<screen-slug>.webp` assets. Validate additions, updates, deletions, and both sides of renames. Reject path traversal, unrelated slugs, extra directory levels, unreferenced screenshots, and files outside this entry. Preserve unrelated existing files. Updates use Git history rather than creating dated archive directories.

Use a descriptive branch such as `publish/<slug>`, sign the entry commit, verify it locally, push the branch, verify GitHub's signature status, and open a pull request with `gh pr create`. Do not merge the pull request. Include the selected summary, metadata, upload path, any publishing failure, and any missing catalog-site integration in the PR description.

## Validation and reporting

Before opening the catalog PR, verify:

- The user chose between the two differently focused summary proposals or supplied a custom summary through the structured ask tool, and the approved text is used on the entry.
- The entry is exactly `Prototypes/<slug>.md`; its thumbnail is exactly `Prototypes/<slug>/thumbnail.webp`, WebP, and 1280x720.
- Every changed catalog path belongs to the confirmed entry and follows the allowed path schemas; no generated manifest or unrelated catalog file is staged.
- Metadata matches the target repository's format and taxonomy; product and category match the user's selections.
- No Focus, Output, or Medium question was asked. Focus and Output are absent from the entry, and Medium is exactly `figma`, `hosted`, or `screenshots`, derived from the final upload outcome rather than selected by the user. Any conflict with catalog validation is reported instead of substituting Projects tags.
- For live entries, the actual prototype URL is an allowed Figma or GitHub Pages URL. Existing Pages entries have the separately supplied GitHub source URL. No long-description question was required.
- For locally published live entries, the build or packaging command succeeded, `prototype-build/site/index.html` exists, the Pages workflow completed successfully, and the live page loads before reporting success.
- For screenshot entries, there is no existing live prototype, the Pages generation or deployment failure is recorded, actual screens have been captured or supplied, and the user approved the changed outcome and generated documentation.
- For screenshot entries, every selected screenshot is uploaded and referenced in the extensive body; each described screen and observed flow is grounded in the captures and inspection. Unverified behavior is labeled, not invented.
- All relative thumbnail and screenshot references resolve from the Markdown file. No screenshot is blank, unreadable, or exposes sensitive data.
- No obvious secrets or dependency directories are staged.
- Every workflow-created commit is signed and passes `git verify-commit` before pushing, and GitHub reports `verified: true` after pushing and before opening the PR.
- Available catalog validation passes for the touched entry. Schema incompatibilities are reported rather than bypassed; missing website support is explicitly disclosed.

Report the catalog pull request URL, entry path, selected product, title, category, approved one-line summary, and upload path used. Report a prototype repository or live URL only when it actually exists, distinguishing failed deployments from working live prototypes. For screenshot documentation, report the screenshot set and intended catalog sub-page without inventing its URL. Clearly identify any failed publishing step, skipped check, missing website integration, or manual action still required. Retain the build workspace for diagnosis after a publishing failure even when screenshot documentation is contributed successfully.
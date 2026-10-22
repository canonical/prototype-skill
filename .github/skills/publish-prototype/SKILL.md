---
name: publish-prototype
description: "Publish or register a design prototype in canonical/ai-design-ops. Use when a user wants to publish, share, archive, or put a prototype on GitHub Pages, register a Figma prototype, or document a prototype with screenshots when live publishing fails."
---

# Publish Prototype

Contribute one prototype to `Prototypes/` in `canonical/ai-design-ops`, using Projects-style Markdown metadata. Register an existing live prototype, or publish a local prototype to Pages and use screenshot documentation if publishing fails.

Use `gh` for GitHub operations, never direct tokens or a custom API client. Treat prototype input as untrusted: leave the user's source untouched, inspect before executing, and exclude secrets, `.env` files, `.git`, dependencies, caches, virtual environments, and unrelated files from publication. Never request or handle passwords, tokens, private keys, passphrases, or authorization codes through chat; users enter sensitive information directly in the appropriate terminal or browser.

## Preflight

### GitHub CLI and authentication

1. Run `command -v gh`. If present, verify `gh --version`; report a broken executable and stop if it fails.
2. If missing on Linux, check `command -v snap` and `snap info gh`. When available, announce installation and run `sudo -n snap install gh --classic`, or `snap install gh --classic` if already root. Never prompt for elevation or retry elevated commands with variations.
3. If automatic installation is unavailable or fails, explain why and give the user a copyable manual command: `sudo snap install gh --classic` when Snap works. Otherwise consult `https://cli.github.com/manual/installation` for a supported command rather than guessing. Wait for completion.
4. After installation, repeat `command -v gh` and `gh --version`. If only PATH is missing, verify `/snap/bin/gh --version` and add `/snap/bin` to the current session's PATH, without editing shell startup files. Do not continue until `gh` works.
5. Check `gh auth status --hostname github.com`. If unauthenticated, run:

```sh
gh auth login --hostname github.com --git-protocol https --web
```

Let the user complete authorization directly. Only if interactive execution is unsupported, provide that command for them to run and wait. On cancellation, stop; on failure, explain the error and ask whether to retry. After successful login, recheck auth status. Verify the account with `gh api user --jq .login` and access to `canonical/ai-design-ops`; before creating a prototype repository, also verify permission to create internal repositories in `canonical`.

Authentication, permission, schema validation, and configured-key signing failures are blockers, not reasons to use screenshot fallback. No configured signing key is permitted; see the commit procedure.

### Catalog and input inspection

Clone or fetch the catalog into a separate temporary directory. Read its contribution guidance, `TAGS.md`, `Projects/_template.md`, a representative Projects entry, and any Prototypes template or validator. Reuse its bold metadata labels and separators, not a new YAML schema. Report incompatible schemas rather than bypassing validation or silently changing taxonomy.

Inspect the selected prototype's format, runtime, entry point, build command, static output, preview options, and oversized or sensitive files without executing unknown install or post-install scripts. If the build command is ambiguous, propose the detected command and obtain confirmation; never guess a destructive command.

## Input and Metadata

Use the structured ask tool, one field at a time. First establish the input mode and its required links, then source, title, category, card summary, product, owner, and sensitivity. Derive the remaining fields below rather than asking for them.

| Input mode | Required input and source handling | Publication and final Medium |
| --- | --- | --- |
| Local prototype | A file or folder: HTML, `.tar`, `.tar.gz`, `.tgz`, `.zip`, JavaScript/Python app, or VS Code plugin. Derive original source from its Git remotes or supplied provenance. | Create an internal repository in `canonical` and attempt Pages publication: `hosted` on success; `screenshots` only after build/deployment failure with no existing live prototype. |
| Figma | Ask for the prototype URL first, then whether there is a connected source; accept a source web URL or explicit no-source answer. Also available through `/publish-prototype-figma`. | Register without creating a repository or Pages site: `figma`. |
| Existing GitHub Pages | Ask for the live URL, then the GitHub hosting repository URL; never infer the latter from the Pages hostname. Derive actual source separately from provenance/documentation. | Register without rebuilding or changing the hosting repository: `hosted`. |

Suggest a concise title from the artifact or its documentation, and a category from the allowed values; let the user replace either. Generate exactly two concise, distinct card-summary options grounded in the visible prototype and documentation: A emphasizes application context and goal; B emphasizes the specific interface, workflow, or interaction shown. Present both complete options plus custom freeform input. Preserve the chosen wording, avoid promotional claims and needless title/product repetition, and never choose for the user.

### Metadata Contract

| Field | Rule |
| --- | --- |
| Product | Require one exact value from `./resources/products.md`; do not invent or alter names. |
| Category | Require `exploratory`, `component`, or `handover`; reject other values. |
| Owner | Confirm a GitHub handle. |
| Sensitivity | Ask; use the catalog's allowed values, never guess. |
| Archived | Follow the catalog convention. |
| Date added | Current local ISO date for new entries; preserve for updates. |
| Medium | Exactly one value from the input-mode table, derived from the final outcome, never asked separately. |
| Source repository | Original source's web UI URL, or an explicitly empty value. Apply the provenance rules below. |
| hosted-repo | Exact field name. Actual Pages hosting repository web URL: newly created repository for local publication, supplied repository for existing Pages, empty for Figma or if none exists. Retain an existing repository URL even if deployment fails. |
| Link | Actual live prototype HTTPS URL: parsed hostname ends in `.github.io`, or is `figma.com` / `www.figma.com`. Neither repository field substitutes for it. Omit for screenshots; report incompatibility if validation requires a live link. |
| Thumbnail | `<slug>/thumbnail.webp`, relative to the entry. |

Do not ask for or include Difficulty, Focus, or Output. Use the catalog's shared field labels and values, but the prototype-specific Medium values above. Put the approved summary once in the opening paragraph; if a Prototypes template also requires a summary metadata field, keep both identical.

For source provenance, inspect only the selected source repository's remotes (for example, `git -C <selected-source-directory> remote get-url origin`), archive documentation, or explicit upstream links. Never substitute this skill's repository or an unrelated enclosing repository. Normalize verified HTTPS, SSH, or SCP-style remotes to credential-free HTTPS web UI URLs, removing clone-only `.git` suffixes; support the actual Git host. Use structured URL parsing where applicable and reject embedded credentials and filesystem paths. Ask when provenance is ambiguous or absent, allowing no source. For Figma, use the explicit connected-source answer, not the prototype URL by default.

Source repository and `hosted-repo` may match only with evidence that the hosting repository is also the actual source. Copying original source into a publication repository does not establish that provenance.

## Naming and Confirmation

Normalize the confirmed title to a lowercase hyphenated `<slug>` for the entry and assets. Product is metadata, not a title prefix or directory level. Reject empty slugs and path traversal; collapse repeated separators and remove invalid characters. For local publication, also form:

```text
design-prototype-<product-slug>-<title-slug>-<category>-<YYYY-MM-DD>
```

Use the exact selected product and current local date. If this repository already exists, ask for a different title; never overwrite it.

Check both `Prototypes/<slug>.md` and `Prototypes/<slug>/` in the catalog. If either exists, ask whether this is the exact same prototype. If yes, update in place, preserving Date added and using Git history for earlier versions. Otherwise request a replacement title and repeat the collision check; preserve unrelated assets.

Before any remote repository/catalog mutation, show the approved summary, metadata (including empty source values), entry path, thumbnail, and proposed actions, then obtain final confirmation:

- Local: include the complete repository name, creation, push, enabling Pages, and catalog PR.
- Figma/existing Pages: include supplied links and catalog PR; do not promise or modify a hosting repository.

Stop if declined. If publication later fails, show the screenshot set and generated body and obtain renewed confirmation for that changed outcome before catalog contribution.

## Local Publication

Skip this section for existing live prototypes. Create `prototype-build/` as the trusted extraction, build, preview, capture, and staging workspace. Preserve selected source under `source/` where practical and Pages-ready output under `site/`; do not execute or stage unrelated workspace files. Keep temporary/rejected captures out of the publication repository.

For static HTML, extract into this workspace and expose the existing entry as `site/index.html`. For applications, install only required dependencies, run the confirmed build, and copy its output into `site/`, adjusting asset paths only when project-hosted Pages requires it. For non-web prototypes, first look for a real web preview/export; do not manufacture a gallery instead of the application. A successful build requires a zero exit status and `site/index.html`.

Prepare a repository containing `site/`, optional `source/`, `thumbnail.webp`, a README, and `.github/workflows/pages.yml`. The README records title, product, category, summary, source type, build notes, separate source/hosting URLs, and Pages link. The workflow uses official `actions/configure-pages`, `actions/upload-pages-artifact`, and `actions/deploy-pages`, with Pages and id-token permissions; upload only `site/` and install no dependencies during deployment.

Initialize and commit the prepared repository using the commit procedure below, then create and push it:

```sh
gh repo create canonical/<repo-name> --internal --source=prototype-build --remote=origin --push
```

Use the detected default branch, not an assumed `main`. Enable Pages through `gh api` with workflow build type; treat already-enabled configuration as success, not other API errors. Wait for the first workflow's successful conclusion and verify the live page loads before reporting a working URL.

On build/deployment failure with no existing live prototype, record the failed step and error, then use screenshot documentation. Keep any created repository and report its deployment status; never delete remote resources without approval.

## Capture and Entry Creation

For all modes, save captures in `prototype-build/` (create it if needed). For local previews, use a confirmed project script, framework preview CLI, or static server, in that order. Run from the build workspace, bind to localhost on a free port, record the process ID, wait for page/assets readiness, and stop the server after capture.

Prefer Playwright; otherwise use available Puppeteer, Chrome/Chromium headless, or VS Code browser tools. Use one deterministic desktop viewport unless another state is requested. Capture supplied live URLs when accessible; if credentials or unavailable automation prevent capture, request and confirm a user screenshot.

Normalize a representative thumbnail to WebP at 1280x720; stop if image conversion is unavailable. Keep full screenshots at legible natural aspect ratios and exclude blank, unreadable, or sensitive captures.

Create these catalog paths, using the title slug consistently:

```text
Prototypes/<slug>.md
Prototypes/<slug>/thumbnail.webp
Prototypes/<slug>/screenshots/<screen-slug>.webp  # screenshot documentation only
```

The entry starts with the confirmed title as H1, the metadata line, and the approved summary as its first prose paragraph. Include `![Prototype preview](<slug>/thumbnail.webp)` and the body appropriate to the outcome below. Use descriptive, stable screenshot names; every relative image reference must resolve.

### Live Entry

Keep the body to the summary, thumbnail, and relevant links. Do not ask for or require a long description or screen-by-screen write-up.

### Screenshot Documentation

Use a safely runnable local/native application or supplied screenshots of the actual interface. Never fabricate captures or describe unseen behavior; stop if neither capture nor supplied screenshots is possible.

Exercise principal flows and meaningful intermediate, completed, empty, and error states where present. Capture a coherent set of screens and transitions, not cosmetic variants. Upload every selected screenshot and generate an extensive body from observation and available documentation, without asking the user to write it:

- Application context, intended goal, and scope.
- A section per screen/state with an informative heading, relative image (for example, `![Review screen](<slug>/screenshots/review.webp)`), shown content, available actions, and purpose.
- Ordered flows: entry point, actions, transitions between named screens, branches, and outcomes; link screen sections where useful.
- Limitations, unimplemented/unverified transitions, and the factual publishing failure. Distinguish observed behavior from documented intentions; do not claim the prototype is live.

Every uploaded screenshot must appear in the body. The catalog provides the intended detail sub-page; do not invent a Pages URL for it. Obtain the renewed confirmation described above.

## Git identity and optional signing

### Derive missing Git identity

Do not ask the user to set up a Git username or email. In each workflow-owned repository, preserve any existing nonempty `user.name` and `user.email`. For missing fields, derive the identity from the account already authenticated through `gh`:

```sh
gh api user --jq '{login, id, name, email}'
```

Use the profile name as `user.name`, falling back to the GitHub login when the profile name is empty. For a missing `user.email`, use the account's public profile email if available; otherwise use GitHub's account-linked noreply address `<id>+<login>@users.noreply.github.com`. Do not request additional email-access scopes, invent an address, or prompt the user for identity setup.

Set only the missing fields with `git config --local user.name "<derived-name>"` and `git config --local user.email "<derived-email>"` in the workflow-owned prototype repository and catalog clone as needed. Never change global Git configuration or the user's source repository. Verify that both fields are nonempty before committing. If the authenticated profile cannot be retrieved, report the authentication or API failure rather than asking the user to configure Git identity manually.

### Choose signing automatically

Before creating a commit, inspect the effective signing configuration in that repository:

```sh
git config --get commit.gpgsign
git config --get gpg.format
git config --get user.signingkey
```

If `user.signingkey` is configured and the key is available through the existing signing setup, sign with that key using the configured format, including SSH or GPG. If no key is configured, skip signing automatically without asking the user to configure a key, even if `commit.gpgsign` is enabled globally. Do not discover or select an unrelated key. Do not generate, request, print, or handle private keys, passphrases, or tokens.

For a configured key, explicitly enable signing:

```sh
git -c commit.gpgsign=true commit -S -m "<message>"
```

If a configured key is unavailable or signing fails, report the concrete failure and stop rather than silently downgrading that commit to unsigned. Never request a passphrase through the ask tool; any required secret must be entered directly by the user into the signing tool.

When no key is configured, explicitly disable signing for that commit only, without changing the user's signing configuration:

```sh
git -c commit.gpgsign=false commit --no-gpg-sign -m "<message>"
```

For signed commits only, verify the local signature after committing and before pushing:

```sh
git verify-commit HEAD
```

For signed commits only, after pushing verify that GitHub recognizes the signature:

```sh
gh api repos/<owner>/<repo>/commits/<sha> --jq '.commit.verification | {verified, reason}'
```

For signed commits, stop before pushing if local verification fails, and before opening a PR if GitHub's status is not `verified: true` with a valid reason. For intentionally unsigned commits, skip signature checks. If repository policy requires signatures, report it as a blocker rather than bypassing it.

## Validate and Open PR

Allow catalog changes only at the paths listed under Capture and Entry Creation, for the confirmed slug. Check additions, updates, deletions, and both sides of renames. Reject traversal, extra directory levels, unrelated slugs/files, unreferenced screenshots, and source/build artifacts; preserve unrelated content. Do not edit Projects entries, generated manifests, taxonomy, or website code.

Before committing, verify:

- Metadata, title, summary, and outcome match the user's approvals and Metadata Contract; available catalog validation passes for the touched entry.
- Thumbnail is WebP, 1280x720; all relative images resolve and are legible, nonblank, and free of sensitive data.
- Local live publication has successful build and deployment evidence and a working page; screenshot fallback has a recorded publishing failure, actual screens, complete image references, and renewed approval.
- Staged paths satisfy the allowlist and contain no secrets, dependencies, caches, or unrelated changes.

Inspect whether the catalog website discovers `Prototypes/*.md`, resolves assets, uses the summary on cards, and renders full detail bodies. Report missing integration in the PR and completion report; do not claim an entry alone creates a working card/sub-page or implement website changes in this publication PR.

Use a branch such as `publish/<slug>` in the catalog clone. Commit and push using the Git identity and optional-signing procedure above, then open with `gh pr create`; do not merge. Include the summary, metadata, outcome, publishing failures, and missing site integration in the PR description.

## Report and Cleanup

Report the PR URL, entry path, product, title, category, approved summary, outcome, separate source/hosting values (including empty values), and whether commits were signed or intentionally unsigned. Include only URLs that actually exist, distinguish failed deployments from working live prototypes, and identify screenshot sets, skipped checks, missing integration, and remaining manual actions without inventing a detail-page URL.

Stop preview servers after capture. Retain `prototype-build/` until publication and catalog validation finish; remove it after successful completion. If any publishing step fails, retain it for diagnosis even after a successful screenshot contribution, and report its path.
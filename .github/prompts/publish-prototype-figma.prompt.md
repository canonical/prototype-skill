---
name: publish-prototype-figma
description: "Register a Figma prototype in the internal prototype registry. Use when the user wants to publish or share a prototype from a Figma URL without packaging local source or creating a GitHub Pages repository."
---

Use the `publish-prototype` skill in Figma input mode.

Ask the user for the Figma URL first. Validate HTTPS and the parsed hostname `figma.com` or `www.figma.com` rather than using a string prefix. Then use the structured ask tool to ask whether there is a connected source, accepting its web URL or an explicit no-source answer. Record that answer as Source repository and leave `hosted-repo` empty.

Follow the skill's GitHub CLI installation and authentication preflight, one-field-at-a-time metadata and summary questions, thumbnail capture, collision checks, and final confirmation. Do not ask for a long description.

Contribute `Prototypes/<slug>.md` and `Prototypes/<slug>/thumbnail.webp` to `canonical/ai-design-ops` using the skill's catalog metadata and validation rules. Do not create a prototype repository or GitHub Pages site for this mode. Open the catalog pull request with `gh`; do not merge it.
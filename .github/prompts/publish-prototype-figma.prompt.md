---
name: publish-prototype-figma
description: "Register a Figma prototype in the internal prototype registry. Use when the user wants to publish or share a prototype from a Figma URL without packaging local source or creating a GitHub Pages repository."
---

Use the `publish-prototype` skill in Figma input mode.

Ask the user for the Figma URL first and validate that it starts with `https://www.figma.com/`. Then ask, one field at a time, for the title, category (`exploratory`, `component`, or `handover`), description, and exact product from `resources/products.md`. Ask for or generate a thumbnail, then show the registry directory and ask for final confirmation before changing GitHub.

Create only the minimal registry directory under `canonical/design-prototypes`:

```text
<product-name>/<prototype-name>/README.md
<product-name>/<prototype-name>/<thumbnail-file>
```

The README may contain the same-directory thumbnail reference, the Figma URL, and an optional source repository URL if the user provides one. Do not create a prototype repository or GitHub Pages site for this mode. Open the registry pull request with `gh`; do not merge it.
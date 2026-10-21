# Prototype Publishing Skill

An agent skill for publishing and registering design prototypes. It accepts local HTML archives, JavaScript or Python apps, VS Code plugins, Figma URLs, and existing GitHub Pages prototypes.

## Import And Use

Ask your agent:

```text
Import the prototype publishing skill from https://github.com/canonical/prototype-skill and use it to publish this prototype.
```

For a Figma prototype, use the shortcut after importing:

```text
/publish-prototype-figma
```

## How It Works

The catalog is `canonical/ai-design-ops`. Each entry is stored as `Prototypes/<slug>.md` with Projects-style metadata and a thumbnail at `Prototypes/<slug>/thumbnail.webp`. The agent proposes two concise card summaries, one emphasizing context and goal and the other the subject shown, and asks the user to choose or write a custom summary.

Existing Figma and GitHub Pages prototypes are registered without asking for a long description. For local prototypes, the skill attempts to build and publish a Pages-ready site in a separate internal repository. If that fails and no live prototype exists, it captures the actual screens, uploads them under `Prototypes/<slug>/screenshots/`, and generates an extensive screen-and-flow description with image references in the entry body. The user confirms this fallback before contribution.

Updates preserve earlier versions through Git history. Catalog website support for the new Prototypes entries, card summaries, assets, and documentation sub-pages must be verified separately; publishing an entry does not itself add that integration.

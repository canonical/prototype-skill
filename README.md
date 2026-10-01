# Prototype Publishing Skill

An agent skill for publishing and registering design prototypes. It accepts local HTML archives, JavaScript or Python apps, VS Code plugins, Figma URLs, and existing GitHub Pages prototypes.

For local prototypes, it can build a Pages-ready site, capture a thumbnail, create an internal GitHub repository, enable GitHub Pages, and submit the prototype to the central registry. The registry stores each prototype under `<product>/<prototype-title>/` with a README and thumbnail. Existing entries can be versioned by archiving the previous README and thumbnail under an ISO-dated archive directory.

## Import And Use

Ask your agent:

```text
Import the prototype publishing skill from https://github.com/canonical/prototype-skill and use it to publish this prototype.
```

For a Figma prototype, use the shortcut after importing:

```text
/publish-prototype-figma
```

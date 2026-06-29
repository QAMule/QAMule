---
name: knowledge-base
description: >
  Manage QAMule knowledge-base files. Use when reading relevant app testing knowledge before Android testing, or when saving, updating, or pruning knowledge discovered during device interaction.
---

# Knowledge Base

QAMule stores reusable Android testing knowledge in `knowledge-base/`.

The agent's `SessionStart` hook discovers `knowledge-base/**/*.md`, reads each file's frontmatter, and injects the available knowledge list into context.

## Use

- Before testing, check the injected knowledge list and read only relevant files with `read_file`.
- If device interaction reveals stable, reusable facts, save them back to `knowledge-base/`.
- If existing knowledge no longer matches app behavior, update or remove it.

## Files

Knowledge files can live anywhere under `knowledge-base/`. A common layout is:

```txt
knowledge-base/
  app/
    overview.md
    screens/
    flows/
  deps/{package}/
```

Each discovered file should have simple frontmatter:

```md
---
name: enable_night_mode
description: "Use when testing night mode, dark theme, appearance settings, or theme switching."
---
```

Keep frontmatter flat and single-line; the hook only parses top-level `key: value` lines. Optional fields like `type`, `scope`, `package`, or `tags` are fine when useful.

## Content Hints

- Overview knowledge often summarizes app scope, common reset steps, major screens or flows, and global testing notes.
- Screen knowledge often includes how to reach the screen, key indicators that confirm the screen, and important elements or selectors.
- Flow knowledge often includes preconditions, screens involved, key action steps, expected result, and common error or recovery paths.
- Quirks usually belong inside the relevant screen or flow. Use a separate quirk file only when the behavior affects multiple screens or flows, or is important enough to reuse independently.
- Dependency knowledge belongs under `knowledge-base/deps/{package}/` when the screen or dialog is owned by another app.

Keep knowledge concise, factual, current, and easy to reuse. Avoid saving temporary observations, one-off failures, or guesses that were not confirmed on device.

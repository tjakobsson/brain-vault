# Vault instructions

Write notes as Obsidian-compatible Markdown. The filename is the note title, so do not repeat it as an H1.

Start each new note with valid YAML frontmatter:

```yaml
---
type: permanent # permanent, literature, or fleeting
status: draft # draft, developing, or established
tags:
  - kebab-case
created: YYYY-MM-DD
---
```

Use `[[Note title]]` for internal links, `[[Note title|label]]` for aliases, and `[[Note title#Heading]]` for sections. Use normal Markdown links for external sources and relative paths for local assets.

Keep the existing metadata and writing style when editing a note.

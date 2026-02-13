---
name: typo3-commit-messages
description: Format commit messages according to TYPO3 commit message rules. Use when writing, suggesting, or reviewing commit messages for TYPO3 projects, or when the user asks for a commit message or TYPO3 commit style.
---

# TYPO3 Commit Messages

Apply these rules when creating or reviewing commit messages for TYPO3 (or this project). Based on [TYPO3 Commit Message rules](https://docs.typo3.org/m/typo3/guide-contributionworkflow/main/en-us/Appendix/CommitMessage.html).

## Summary line (first line)

- Start with a **keyword**: `[BUGFIX]`, `[FEATURE]`, `[TASK]`, `[DOCS]`
- Breaking changes: put `[!!!]` before the keyword, e.g. `[!!!][FEATURE]`
- Max 52 characters for the subject (72 absolute max)
- **Imperative mood** (“Add feature”, not “Added feature”).  
  Test: “If applied, this commit will **<Subject>**” must read naturally
- Capitalize the word after the keyword

## Examples

```
[TASK] Add initial TYPO3 Agent Kit boilerplate
[FEATURE] Add option to hide BE search box in list module
[BUGFIX] Fix backend edit URL in admin panel
[DOCS] Add documentation for version 9.1
```

## Body (optional)

- Describe what changed, briefly
- Use `*` for bullet points and a hanging indent
- Wrap lines at 72 characters

## Relationships (optional; required for TYPO3 Forge)

- `Resolves: #12345` – closes a Forge ticket
- `Related: #12340` – links another ticket
- `Releases: main, 13.4` – target branches

In own repos without Forge, Resolves/Releases can be omitted.

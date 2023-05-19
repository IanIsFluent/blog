---
tags:
  - typescript
  - vscode
---

# Organise Usings on Save

If your linter is complaining about unused usings or import sort order, you can make it stop by organising your usings using the shift+alt+o shortcut in VSCode.

But to save some typing, you can just get VSCode to do this for you on every save by adding the following to your settings.json:

```json
"editor.codeActionsOnSave": {
  "source.organizeImports": true
}
```

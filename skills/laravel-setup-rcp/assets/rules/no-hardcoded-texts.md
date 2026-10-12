---
paths:
  - 'app/**'
  - 'resources/views/**'
  - 'lang/**'
---

# No hardcoded texts

Always use lang folder to place the texts and translations.

Don't use:

```
__('Create team')
```

Use instead (for example):

```
__('teams.create')
```

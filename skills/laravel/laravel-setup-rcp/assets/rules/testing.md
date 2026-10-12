---
paths:
  - '**'
---

# Testing guidelines

- **Always run the full test suite** using `composer test` after completing any request or making changes to the codebase. It runs lint checks, PHPStan and Pest.
- A task is complete only when `composer test` is green. If it fails, fix the issues before finalizing your response.

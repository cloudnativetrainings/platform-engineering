---
name: code-linter
description: Checks a lab README for typos and grammar issues. Invoke when the user says "lint code".
---

# Code linter

Check the lab `README.md` files for issues within the code blocks. Furthermore check other code files, eg yaml files, for issues within the code.

- If the user names a specific lab, check only that one.
- Otherwise check every top-level `*/README.md` (skip `.99_todos/`).
- Report findings as `file_path:line_number` with the fix suggestion.
- Ask before changing files if this should be done

## Must not

- Do not read, open, or grep files under `.secrets/` — passwords, SSH keys, service-account JSON are off-limits.
- Do not read, open, or grep files under `.99_todos/` 
- Do not "fix" placeholders (`TODO`, `XXXXX`, `TODO-STUDENT-EMAIL@...`) — they are intentional.
- Ignore grammar or typo issues within prose texts in md files. 
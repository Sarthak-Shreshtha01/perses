# Contribution & Git Workflow

This guide details the exact steps and checks required before submitting a Pull Request to Perses.

---

## 1. DCO (Developer Certificate of Origin) Sign-Off

The Perses project requires every commit to have a DCO sign-off.

### How to Sign Off:
Always commit with the `-s` flag:
```bash
git commit -s -m "[BUGFIX] Fix alignment in project card title"
```

This automatically appends:
```text
Signed-off-by: Your Name <your.email@example.com>
```

> ⚠️ **Warning:** If your PR fails the DCO check on GitHub, you will need to rebase and sign off commits using:
> `git rebase --signoff HEAD~N` and force push (`git push --force-with-lease`).

---

## 2. Commit Message & PR Title Naming Conventions

Perses generates its changelog directly from commit and PR titles. Therefore, they **must** begin with one of the following catalog tags:

| Tag | Purpose | Example |
| :--- | :--- | :--- |
| `[BUGFIX]` | Fixing a bug or unexpected behavior | `[BUGFIX] ensure project selector preserves active tab` |
| `[ENHANCEMENT]` | Improving an existing feature or UI experience | `[ENHANCEMENT] add search filter to datasource list` |
| `[FEATURE]` | Adding an entirely new feature | `[FEATURE] support bulk deletion of dashboards` |
| `[DOC]` | Updating documentation or READMEs | `[DOC] update UI architecture diagram links` |
| `[BREAKINGCHANGE]` | Breaking API or stored resource changes | `[BREAKINGCHANGE] remove deprecated layout field` |
| `[IGNORE]` | Internal tooling/CI changes (omitted from changelog) | `[IGNORE] upgrade oxlint and oxfmt version` |

---

## 3. Pre-Flight Quality Checklist

Before committing code or opening a PR, run these commands inside the `ui/` folder and ensure all pass cleanly:

```bash
cd ui

# 1. Check for Oxlint and React Doctor issues
npm run lint

# 2. Check Oxfmt code formatting
npm run format:check

# If format check fails, automatically format with:
# npm run format

# 3. Verify TypeScript types
npm run type-check

# 4. Run Vitest test suite
npm run test
```

### Oxlint Rules:
- Do not disable lint rules across files.
- If a narrow exception is required, use `// oxlint-disable-next-line <rule-name>` accompanied by an explanatory comment.

---

## 4. Submitting Your Pull Request

1. **Push to your fork branch:**
   ```bash
   git push -u origin your-feature-branch
   ```
2. **Open PR on GitHub:**
   - Set the title to match the commit convention (e.g. `[BUGFIX] Fix alignment in project card title`).
   - Reference the issue number in the PR description: `Fixes #1234` or `Closes #1234`.
   - Provide a concise description of what changed, why it changed, and how it was tested.
   - Include before/after screenshots or GIFs for any user-facing UI changes!

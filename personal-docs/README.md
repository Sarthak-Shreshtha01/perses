# Perses Personal Contributor Notes & Guide

Welcome to your personal onboarding and development notes for contributing to **Perses**.

---

## 📚 Table of Contents

1. [Running Perses Locally & Usage Guide](./running-perses-locally.md)
   - Step-by-step instructions to run frontend & backend, communication architecture, and a guided tour on how to use Perses.
2. [Environment Setup Guide](./environment-setup.md)
   - Local Node.js / npm requirements, Windows tips, backend setup (Docker / Go).
3. [UI Guidelines & Best Practices](./ui-guidelines-and-conventions.md)
   - Tech stack, directory structure, MUI styling rules, TypeScript rules, React standards.
4. [Contribution & Git Workflow](./contribution-and-git-workflow.md)
   - DCO commit sign-offs (`-s`), PR title tags (`[BUGFIX]`, `[FEATURE]`), pre-flight checks.
5. [Architecture & Repo Map](./repo-architecture-map.md)
   - Monorepo boundaries (`perses/perses` vs `perses/shared` vs `perses/spec`), package responsibilities.
6. [Finding & Fixing UI Issues](./finding-and-fixing-ui-issues.md)
   - Where to find issues, how to locate relevant code in `ui/app`, debugging techniques, writing tests.

---

## ⚡ Quick Command Cheat Sheet

Always run frontend commands from the `ui/` folder:

```bash
cd ui

# 1. Install dependencies
npm ci

# 2. Run local dev server (App on http://localhost:3000)
npm run start

# 3. Quality & validation commands
npm run lint           # Oxlint + React Doctor
npm run format:check   # Oxfmt check
npm run format         # Auto-format files with Oxfmt
npm run type-check     # TypeScript check (tsc --noEmit)
npm run test           # Vitest unit & integration tests
```

---

## 📌 Critical Golden Rules

1. **Commit Sign-Off:** Always use `git commit -s` (DCO requirement).
2. **Commit Naming:** Prefix commits and PR titles with tags like `[BUGFIX] <message>` or `[ENHANCEMENT] <message>`.
3. **No Hardcoded Styles:** Use Material UI theme tokens in `sx` props, never raw hex colors or arbitrary px values.
4. **Strict TypeScript:** Never use `any`; use `unknown` and type narrowing.
5. **Direct Icon Imports:** Always `import FooIcon from 'mdi-material-ui/Foo'`, not from root barrel.

# Repository Architecture Map

Understanding how Perses repositories and packages interact prevents editing files in the wrong layer or looking for code in the wrong place.

---

## 1. Multi-Repository Ecosystem

Perses features are split across 4 GitHub repositories:

```text
       ┌────────────────────────┐
       │      perses/spec       │   Canonical types (@perses-dev/spec)
       └───────────┬────────────┘
                   │
       ┌───────────▼────────────┐
       │     perses/shared      │   Shared libraries (@perses-dev/components,
       └───────────┬────────────┘   dashboards, explore, plugin-system, client)
                   │
        ┌──────────┴──────────┐
        ▼                     ▼
┌───────────────┐     ┌───────────────┐
│ perses/perses │     │perses/plugins │  (TimeSeries, Gauge, Prometheus, etc.)
│  (THIS REPO)  │     └───────────────┘
└───────────────┘
```

### Package Roles

| Package | Repository | Role |
| :--- | :--- | :--- |
| **`@perses-dev/app`** | `perses/perses` (*this repo*) | The main React single-page app: routes, page layout, auth, admin, project listings, REST integration. |
| **`@perses-dev/components`** | `perses/shared` | Core UI widgets: dialogs, tables, Monaco/CodeMirror editors, formatting controls. |
| **`@perses-dev/dashboards`** | `perses/shared` | Dashboard viewing, grid layouts, panel containers, dashboard edit mode. |
| **`@perses-dev/explore`** | `perses/shared` | The Explore querying interface. |
| **`@perses-dev/plugin-system`**| `perses/shared` | Plugin registry, runtime variables, datasource resolvers, module federation host. |
| **`@perses-dev/spec`** | `perses/spec` | Data contracts & schemas for panels, queries, variables, datasources. |
| **Plugin Packages** | `perses/plugins` | Individual panel and query plugins (e.g. `@perses-dev/prometheus-plugin`). |

---

## 2. In-Depth: Inside `perses/perses` (This Repository)

This repo contains both backend and frontend orchestration:

```text
perses/
├── cmd/
│   ├── perses/            # Backend server binary entry point
│   └── percli/            # CLI client binary
├── internal/api/          # Go API routing, database persistence, handlers
├── pkg/model/             # Go resource models and validation
├── dev/                   # Dev configuration files and seed data
├── docs/                  # Documentation (built via MkDocs)
└── ui/                    # FRONTEND MONOREPO ROOT
    ├── app/               # Main application package (@perses-dev/app)
    ├── e2e/               # Playwright browser end-to-end test suite
    ├── internal-utils/    # Internal build/test helpers
    └── package.json       # Monorepo dependencies (Turbo, Oxlint, Oxfmt, Vitest)
```

---

## 3. Working with `perses/shared` Locally (Advanced)

If a UI issue or feature spans across both the app shell (here) and a shared library (like `@perses-dev/components` or `@perses-dev/dashboards`):
1. Clone `perses/shared` as a sibling directory (`../shared`).
2. Follow linking instructions in the `perses/shared` README.
3. In `ui/app`, run:
   ```bash
   npm run start:shared
   ```
   This compiles and links the local `shared` code with hot-reloading.

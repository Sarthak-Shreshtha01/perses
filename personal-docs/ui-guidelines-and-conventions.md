# UI Guidelines & Coding Standards

This guide summarizes the frontend design standards, styling conventions, and TypeScript/React patterns required by Perses maintainers.

---

## 1. Directory Structure (`ui/app/src/`)

```text
ui/app/src/
├── components/     # Global, reusable UI components (Dialogs, Header, Breadcrumbs, Lists, etc.)
├── views/          # Route-level feature views (Admin, Projects, Auth, Config, Home, etc.)
│   ├── admin/      # Admin pages & management sub-views
│   ├── projects/   # Project & dashboard listings
│   └── ...
├── context/        # Application-wide React contexts
├── model/          # Data models, API query hooks (TanStack Query), formatters, validators
├── validation/     # Zod schemas for forms and API payloads
├── guard/          # Route guards (Authentication / RBAC protection)
├── utils/          # General utility functions
└── Router.tsx      # Top-level client-side routing definitions
```

### Module Organization Principles
- **Keep feature code local:** Components and hooks used only by a single feature stay inside that feature's folder (e.g., `views/admin/users/`).
- **Shared threshold:** Only promote a component to `src/components/` if it is genuinely used in **3 or more distinct places**.
- **File size rule of thumb:** If a file exceeds ~300 lines, consider breaking out subcomponents or helper functions into co-located files.

---

## 2. Styling Standards (Material UI & Theme Tokens)

Perses uses **Material UI v6 (`@mui/material`)**.

### The Golden Rule: Use Theme Tokens
**Never** hardcode hex colors (`#fff`, `#1976d2`) or arbitrary pixel numbers (`margin: 14px`). Always reference theme values:

```tsx
// ❌ WRONG: Hardcoded values
<Box sx={{ backgroundColor: '#f0f0f0', padding: '15px', color: '#111' }}>

// ✅ CORRECT: Theme tokens
<Box
  sx={(theme) => ({
    backgroundColor: theme.palette.background.default,
    padding: theme.spacing(2),
    color: theme.palette.text.primary,
  })}
>
```

### Icon Imports
Never import icons from the `mdi-material-ui` root barrel. Import directly from the specific module:

```tsx
// ❌ WRONG: Bundle-bloating barrel import
import { Pencil, Delete } from 'mdi-material-ui';

// ✅ CORRECT: Direct module imports
import PencilIcon from 'mdi-material-ui/Pencil';
import DeleteIcon from 'mdi-material-ui/Delete';
```

---

## 3. TypeScript Guidelines

Perses uses strict TypeScript configuration. Avoid escape hatches:

1. **No `any`:**
   Use `unknown` coupled with type guards or schema validation (e.g., Zod).
2. **Explicit Props Interface:**
   Name component props using `ComponentNameProps` and export them:
   ```tsx
   export interface UserCardProps {
     userId: string;
     onSelect?: (id: string) => void;
   }

   export const UserCard: React.FC<UserCardProps> = ({ userId, onSelect }) => { ... };
   ```
3. **Named Exports Only:**
   Avoid `export default`. Use named exports everywhere:
   ```tsx
   export function ProjectHeader() { ... }
   ```
4. **Avoid `null`:**
   Prefer optional fields (`prop?: string`) and `undefined`. Reserve `null` strictly for domains where backend APIs explicitly represent a null value.

---

## 4. React Standards & Performance

1. **Pure Rendering:**
   - Never mutate props, state, or refs during render.
   - Do not read or write `ref.current` inside the render body — only in effects, layout effects, or event handlers.
2. **Derive State During Render:**
   - Do not use `useEffect` to copy props or server data into local state. Compute derived values directly in the render function or with `useMemo`.
3. **Memoization & Context:**
   - Memoize React context values (`useMemo`) when passing objects or functions so consumers do not re-render unnecessarily.
4. **State Management Layers:**
   - **Server State:** Use `@tanstack/react-query` hooks (found in `src/model/`).
   - **Complex Client State:** Use [Zustand](https://github.com/pmndrs/zustand) with Immer.
   - **Local UI State:** Standard `useState`.

---

## 5. Testing Patterns (Vitest + React Testing Library)

- Place tests beside the source code: `MyComponent.test.tsx`.
- Test observable user behavior rather than implementation details:
  ```tsx
  // ✅ Good: Accessible role-based query
  const button = screen.getByRole('button', { name: /save project/i });
  await userEvent.click(button);

  // ❌ Avoid: data-testid or CSS class queries unless unavoidable
  const button = screen.getByTestId('save-btn');
  ```

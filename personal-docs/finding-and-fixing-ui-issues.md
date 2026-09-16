# Finding & Fixing Minor UI Issues

This guide helps you identify, reproduce, and fix minor UI issues effectively as a beginner in the Perses codebase.

---

## 1. Where to Find Issues to Work On

1. **GitHub Issues**:
   - Filter by:
     - `label:area/ui`
     - `label:"good first issue"`
     - `label:bug`
   - Link: [Perses Open UI Issues](https://github.com/perses/perses/issues?q=is%3Aissue+is%3Aopen+label%3Aarea%2Fui)
2. **Comment Before Coding**:
   - Post a polite comment on the issue: *"Hi, I'd like to work on this issue if it's available!"*
   - Wait for a maintainer to assign it to avoid duplicate effort.

---

## 2. Common Categories of Minor UI Issues

Minor UI issues in Perses typically fall into these areas:

1. **Alignment, Padding & Spacing Quirks**:
   - Dialog margins, button alignments in toolbars, table header misalignments.
   - *Where it lives:* Check `ui/app/src/components/` (e.g. `dialogs/`, `page-header/`, `datagrid.tsx`).
2. **Form Validation & Error States**:
   - Missing helper text, unhandled edge cases in form submission, or missing error feedback.
   - *Where it lives:* Check `ui/app/src/validation/` (Zod schemas) and form views under `ui/app/src/views/`.
3. **Empty States & Loading States**:
   - Table or list showing a blank screen instead of a friendly `EmptyState` component when search results are empty.
   - *Where it lives:* Check `ui/app/src/components/EmptyState/`.
4. **Accessibility (a11y) & Keyboard Navigation**:
   - Missing `aria-label`s on icon-only buttons, broken tab order, or unhandled Escape key behaviors on modals.
   - *Where it lives:* Buttons in `Header`, `ShortcutHelpModal`, and navigation menus.
5. **Localization (i18n)**:
   - Hardcoded strings instead of using `useTranslation()` from `react-i18next`.
   - *Where it lives:* `ui/app/src/locales/`.

---

## 3. How to Trace Code from the Browser

When reproducing an issue in the browser:
1. **Look at the URL Route**:
   - Check `ui/app/src/Router.tsx` to find the React component mapped to that path.
   - Example: `/projects/:projectName` maps to `ProjectView` in `ui/app/src/views/projects/`.
2. **Use React Developer Tools**:
   - Inspect the component tree to see the exact component name (e.g., `DashboardCard`, `VariableEditor`).
3. **Search the Text**:
   - Grep for unique UI labels or buttons in `ui/app/src/`.

---

## 4. Testing Your Fix

Always add or update a unit test when fixing a bug!

### Example: Writing a Vitest test for a UI Component
```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MyComponent } from './MyComponent';

describe('MyComponent', () => {
  it('should render correct label and handle click', async () => {
    const handleAction = vi.fn();
    render(<MyComponent onAction={handleAction} />);

    const button = screen.getByRole('button', { name: /confirm/i });
    expect(button).toBeInTheDocument();

    await userEvent.click(button);
    expect(handleAction).toHaveBeenCalledTimes(1);
  });
});
```

Run only your specific test while iterating:
```bash
# In ui/ folder:
npx vitest run src/components/MyComponent.test.tsx
```

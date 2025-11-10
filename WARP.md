# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is Chromatic's UI Testing Handbook React template - a Storybook-based project demonstrating component-driven development and UI testing patterns. It's a task management application ("Taskbox") built with React 18, Vite, and Storybook 8.

## Commands

### Development
```bash
# Start Vite development server (port 3000)
yarn dev
# or
pnpm dev

# Start Storybook (port 6006)
yarn storybook
# or
pnpm storybook
```

### Build
```bash
# Build production app
yarn build

# Build Storybook static site
yarn build-storybook
```

### Testing
```bash
# Run Storybook test runner (requires Storybook to be running)
yarn test-storybook

# Run linter
yarn lint
```

### MSW Setup
```bash
# Initialize Mock Service Worker (only needed once)
yarn init-msw
```

### Visual Testing
```bash
# Run Chromatic for visual regression testing
yarn chromatic
```

## Architecture

### Component Structure

The application follows a **bottom-up component composition** pattern:

1. **Atomic Components** (`src/components/`)
   - `Task.jsx` - Individual task item with archive/pin/edit capabilities
   - `TaskList.jsx` - List container that handles loading, empty, and populated states

2. **Screen Components** (`src/`)
   - `LoginScreen.jsx` - Authentication screen with form handling
   - `InboxScreen.jsx` - Main task management screen, composes TaskList with data fetching
   - `App.jsx` - Root component managing authentication state and routing

### State Management

**Custom Hooks with useReducer:**
- `useTasks.js` - Task management (fetch, archive, pin, edit, delete)
- `useAuth.js` - Authentication state (login/logout)

Both hooks use the reducer pattern for predictable state updates and expose:
- Current state
- Dispatch function for actions

### Data Flow

```
App (auth gate)
  ├─> LoginScreen (when logged out)
  └─> InboxScreen (when logged in)
      └─> TaskList (receives tasks from useTasks hook)
          └─> Task (individual items)
```

Task actions flow up through callbacks:
- `onArchiveTask`, `onTogglePinTask`, `onEditTitle` → dispatch to reducer

### API Integration

- **Data Fetching:** Uses fetch API with AbortController for cleanup
- **Mocking:** MSW (Mock Service Worker) configured for Storybook stories
- **Endpoints:**
  - `GET /tasks` - Fetch task list
  - `POST /authenticate` - User authentication

### Storybook Integration

**Key Features:**
- Stories colocated with components (`*.stories.jsx`)
- MSW addon for API mocking in stories
- Interaction testing using `@storybook/test` (play functions)
- Stories serve as both documentation and test fixtures

**Story Pattern:**
```javascript
export default {
  component: Component,
  title: 'ComponentName',
};

export const StoryName = {
  args: { /* component props */ },
  parameters: {
    msw: { handlers: [/* MSW handlers */] }
  },
  play: async ({ canvasElement }) => {
    // Interaction tests
  },
};
```

### Styling
- Global styles in `src/index.css`
- Component styles use className-based CSS (no CSS modules)
- Icon system uses CSS classes prefixed with `icon-`

## Development Patterns

### Adding New Components

1. Create component file in appropriate directory (`src/components/` for reusable, `src/` for screens)
2. Create corresponding `.stories.jsx` file for Storybook documentation
3. Define PropTypes for type checking
4. Export stories with various states (Default, Loading, Error, Edge cases)

### Task State Machine

Tasks have three states:
- `TASK_INBOX` - Default state
- `TASK_PINNED` - Pinned tasks (appear first in list)
- `TASK_ARCHIVED` - Archived tasks (hidden from main list)

TaskList automatically sorts: pinned tasks first, then inbox tasks.

### Testing with Play Functions

Use Storybook's play functions for interaction testing:
```javascript
play: async ({ canvasElement }) => {
  const canvas = within(canvasElement);
  const element = await canvas.findByRole('button');
  await userEvent.click(element);
  await expect(element).toHaveAttribute('aria-label', 'expected');
}
```

## Important Notes

- **Package Manager:** Project supports both Yarn and pnpm (uses pnpm-lock.yaml)
- **Email Input Bug:** LoginScreen form has incorrect input name/id mapping (input has no `id` attribute but label references it)
- **MSW Configuration:** Worker files must be in `public/` directory for proper functioning
- **Storybook Config:** Stories auto-discovered from `src/**/*.stories.@(js|jsx|ts|tsx)`

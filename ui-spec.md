# UI Specification

Front-end contract: screens, design tokens, wireframes, state, accessibility, theming, and loading/error/empty states.

## Screens

### 1. Login Screen
**Route:** `/login`
**Purpose:** User authentication through Keycloak OpenID Connect flow. Users authenticate to access the todo application with role-based permissions.

**Components:**
- Application header with white-label logo and brand name
- Login form with Keycloak authentication button
- Error message display area for authentication failures
- Loading spinner during authentication redirect
- Footer with support links and legal information

**Interactions:**
- Click login button initiates Keycloak Authorization Code Flow
- Successful authentication redirects to dashboard with JWT token
- Failed authentication displays error message with retry option
- Token stored in browser localStorage for subsequent requests

---

### 2. Dashboard Screen
**Route:** `/` (authenticated home)
**Purpose:** Main landing page displaying todo statistics, quick actions, and navigation to other sections. Provides overview of user's task management.

**Components:**
- User profile header with avatar, username, and logout button
- Statistics cards showing total todos, completed, pending, and deleted
- Quick action buttons for creating new todo and viewing all todos
- Recent todos list showing last 5 modified items
- Navigation sidebar or tabs for list, settings, and profile
- Loading indicator for statistics data
- Error banner for failed data fetches

**Interactions:**
- Click logout clears token and redirects to login screen
- Statistics cards update in real-time as todos change
- Quick action buttons navigate to respective screens
- Recent todo items are clickable to view detail
- Navigation tabs switch between main application sections

---

### 3. Todo List Screen
**Route:** `/todos`
**Purpose:** Display all todos with filtering, sorting, and search capabilities. Primary interface for managing todo items.

**Components:**
- Page header with title and filter controls
- Search input field for filtering todos by title text
- Filter dropdown for status (All, Completed, Pending, Local, Remote)
- Sort dropdown for ordering (ID, Title, Status, Date Created)
- Todo form with text input and add button (inline or modal)
- Todo list with pagination for large datasets
- Empty state illustration and message when no todos match filters
- Loading skeleton or spinner during data fetch
- Error banner for failed API calls
- Batch action buttons (delete multiple, mark all completed)

**Interactions:**
- Search input filters list in real-time on keyup or submit
- Filter dropdown reduces list to matching status
- Sort dropdown reorders list without API call (client-side)
- Add todo form creates new item with title and optional completed status
- Todo items display id, title, completion status, and delete button
- Delete button removes todo with confirmation modal
- Pagination controls navigate through pages
- Batch actions apply to selected checkboxes

---

### 4. Todo Detail Screen
**Route:** `/todos/:id`
**Purpose:** View and edit individual todo details with full information display and modification capabilities.

**Components:**
- Breadcrumb navigation back to list
- Todo detail header with id and status badge
- Title display with edit capability (inline or form)
- User information display (userId)
- Creation and modification timestamps
- Completion status toggle switch
- Source indicator (Local or Remote/JSONPlaceholder)
- Action buttons: Save, Cancel, Delete
- Loading state when fetching todo details
- Error message for failed fetch or update
- Delete confirmation modal

**Interactions:**
- Title field allows inline editing or opens edit mode
- Completion toggle switches between true and false
- Save button updates todo via PATCH endpoint
- Cancel button reverts unsaved changes
- Delete button opens confirmation modal
- Confirm delete removes todo and redirects to list
- Breadcrumb click returns to previous list view with filters preserved

---

### 5. Create/Edit Todo Screen
**Route:** `/todos/new` (create) or `/todos/:id/edit` (edit)
**Purpose:** Dedicated screen for creating new todos or editing existing ones with comprehensive form validation.

**Components:**
- Page header with title (Create Todo or Edit Todo #id)
- Back button to return to previous screen
- Form field: Title (required, text input, placeholder text)
- Form field: Completed status (checkbox or toggle)
- Form field: Notes or description (optional, textarea)
- Form field: Due date (optional, date picker)
- Validation error messages per field
- Submit button (Create Todo or Save Changes)
- Cancel button to abandon changes
- Loading spinner during form submission
- Success notification after successful create/update
- Error banner for submission failures

**Interactions:**
- Title field validates non-empty after trimming whitespace
- Completed checkbox defaults to unchecked for new todos
- Form submission sends POST or PATCH request to backend
- Loading state disables submit button and form fields
- Success notification auto-dismisses after 3 seconds
- Cancel button prompts for confirmation if unsaved changes exist
- Back button without changes returns immediately
- Validation errors display inline below respective fields

---

### 6. Settings Screen
**Route:** `/settings`
**Purpose:** Application configuration interface for user preferences, theme selection, and account management options.

**Components:**
- Page header with Settings title
- Navigation tabs or sections: Appearance, Account, Notifications, Data
- Appearance section: Theme selector (Light, Dark, System), Font size slider
- Account section: Profile information (read-only), Change password form
- Notifications section: Email notifications toggle, In-app notifications toggle
- Data section: Export todos button, Import todos file upload, Clear all data button
- Save button for persisting preferences
- Loading states for data operations
- Success/error notifications for settings changes
- Confirmation modal for destructive actions (clear data)

**Interactions:**
- Theme selector updates CSS variables and persists preference
- Font size slider adjusts root font-size CSS property
- Change password form validates and sends update to Keycloak
- Export button downloads todos as JSON file
- Import file accepts JSON upload and validates format
- Clear all data button requires confirmation before executing
- Save button persists local preferences to localStorage
- Preferences apply immediately without page reload

---

### 7. Profile Screen
**Route:** `/profile`
**Purpose:** Display and manage user profile information, authentication status, and session details.

**Components:**
- User profile header with avatar image or initials
- Display name field (read-only from Keycloak)
- Username field (read-only)
- Email address field (read-only)
- User roles display (todo:read, todo:write, todo:delete, todo:admin)
- Session information: Login time, Token expiration
- Action buttons: Refresh token, View Keycloak profile, Logout
- Token display section (expandable) with copy button
- Last activity timestamp
- Logout confirmation modal
- Loading indicators during token refresh

**Interactions:**
- Refresh token button sends silent request to Keycloak
- View Keycloak profile opens external link to Keycloak account page
- Logout button clears localStorage and redirects to login
- Copy token button copies JWT to clipboard
- Token section expands/collapses to show full token
- Roles display shows badges for each assigned role
- All fields are read-only as they come from Keycloak identity provider
- Session info updates in real-time every 60 seconds

---

### 8. Error/Empty States Screen
**Route:** `/error` (global error) or per-screen states
**Purpose:** Unified handling of error states, empty states, and loading states across the application with consistent UX patterns.

**Global 404 Error Screen:**
- Illustration or icon for page not found
- Error message: "Page not found"
- Description: "The page you're looking for doesn't exist or has been moved."
- Action button: "Go to Dashboard" or "Go Home"
- Support link: "Contact support"

**Global 500 Error Screen:**
- Illustration or icon for server error
- Error message: "Something went wrong"
- Description: "We encountered an unexpected error. Please try again later."
- Action buttons: "Retry" (reloads current page), "Go to Dashboard"
- Error ID display for support reference

**Per-Screen Loading States:**
- Skeleton loaders with animated shimmer effect
- Spinner indicator with loading text
- Progress bar for long-running operations
- Cancel button for interruptible operations

**Per-Screen Error States:**
- Error banner with icon and message
- Retry button to attempt failed operation again
- Dismiss button to close error banner
- Error details expandable section for debugging

**Per-Screen Empty States:**
- Empty list illustration or icon
- Empty state message specific to context
- Call-to-action button (e.g., "Create your first todo")
- Helpful hint text explaining why list is empty

## Design Tokens

### Color Tokens (CSS Variables)

```css
/* Primary Colors */
--brand-primary: #6366f1;          /* Main brand color, white-label configurable */
--brand-primary-hover: #4f46e5;    /* Darker shade for hover states */
--brand-primary-light: #e0e7ff;    /* Light shade for backgrounds and tints */
--brand-primary-dark: #312e81;     /* Darkest shade for emphasis */

/* Semantic Colors */
--color-success: #10b981;           /* Green for success, completed states */
--color-success-light: #d1fae5;     /* Light green for backgrounds */
--color-warning: #f59e0b;           /* Amber for warnings */
--color-warning-light: #fef3c7;     /* Light amber for backgrounds */
--color-error: #ef4444;             /* Red for errors, delete actions */
--color-error-light: #fee2e2;       /* Light red for error backgrounds */
--color-info: #3b82f6;              /* Blue for informational messages */
--color-info-light: #dbeafe;        /* Light blue for info backgrounds */

### Neutral Colors */
--color-gray-50: #f9fafb;          /* Lightest gray, page backgrounds */
--color-gray-100: #f3f4f6;         /* Very light gray, card backgrounds */
--color-gray-200: #e5e7eb;         /* Light gray, borders */
--color-gray-300: #d1d5db;         /* Medium light gray, disabled text */
--color-gray-400: #9ca3af;         /* Medium gray, placeholders */
--color-gray-500: #6b7280;         /* Gray, secondary text */
--color-gray-600: #4b5563;         /* Dark gray, body text */
--color-gray-700: #374151;         /* Darker gray, headings */
--color-gray-800: #1f2937;         /* Dark gray, primary text */
--color-gray-900: #111827;         /* Darkest gray, black text */

/* White-Label Variables */
--brand-primary: #6366f1;          /* Override for custom brand color */
--brand-secondary: #8b5cf6;         /* Secondary brand accent color */
--brand-logo-url: url('/logo.svg'); /* Override for custom logo */
--brand-name: 'Todo App';           /* Override for custom product name */

/* Theme Mode Variables */
--background-primary: var(--color-gray-50);
--background-card: #ffffff;
--background-input: #ffffff;
--text-primary: var(--color-gray-900);
--text-secondary: var(--color-gray-600);
--border-color: var(--color-gray-200);

/* Dark Theme Overrides (when --theme-mode = dark) */
--background-primary: #111827;
--background-card: #1f2937;
--background-input: #374151;
--text-primary: #f9fafb;
--text-secondary: #9ca3af;
--border-color: #374151;
```

### Typography Tokens

```css
/* Font Families */
--font-family-base: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
--font-family-mono: 'SFMono-Regular', Consolas, 'Liberation Mono', Menlo, monospace;

/* Font Sizes */
--font-size-xs: 0.75rem;           /* 12px */
--font-size-sm: 0.875rem;          /* 14px */
--font-size-base: 1rem;            /* 16px */
--font-size-lg: 1.125rem;          /* 18px */
--font-size-xl: 1.25rem;           /* 20px */
--font-size-2xl: 1.5rem;           /* 24px */
--font-size-3xl: 1.875rem;         /* 30px */
--font-size-4xl: 2.25rem;          /* 36px */

/* Font Weights */
--font-weight-normal: 400;
--font-weight-medium: 500;
--font-weight-semibold: 600;
--font-weight-bold: 700;

/* Line Heights */
--line-height-tight: 1.25;
--line-height-normal: 1.5;
--line-height-relaxed: 1.75;

/* Letter Spacing */
--letter-spacing-tight: -0.025em;
--letter-spacing-normal: 0;
--letter-spacing-wide: 0.025em;
```

### Spacing Tokens

```css
--spacing-0: 0;
--spacing-1: 0.25rem;   /* 4px */
--spacing-2: 0.5rem;    /* 8px */
--spacing-3: 0.75rem;   /* 12px */
--spacing-4: 1rem;      /* 16px */
--spacing-5: 1.25rem;   /* 20px */
--spacing-6: 1.5rem;    /* 24px */
--spacing-8: 2rem;      /* 32px */
--spacing-10: 2.5rem;   /* 40px */
--spacing-12: 3rem;     /* 48px */
--spacing-16: 4rem;     /* 64px */
--spacing-20: 5rem;     /* 80px */
--spacing-24: 6rem;     /* 96px */
```

### Border Radius Tokens

```css
--radius-none: 0;
--radius-sm: 0.25rem;   /* 4px */
--radius-base: 0.5rem;  /* 8px */
--radius-md: 0.625rem;   /* 10px */
--radius-lg: 0.75rem;    /* 12px */
--radius-xl: 1rem;       /* 16px */
--radius-full: 9999px;
```

### Shadow Tokens

```css
--shadow-xs: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
--shadow-sm: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px -1px rgba(0, 0, 0, 0.1);
--shadow-base: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
--shadow-md: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -4px rgba(0, 0, 0, 0.1);
--shadow-lg: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
--shadow-xl: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
```

### Transition Tokens

```css
--transition-fast: 150ms ease-in-out;
--transition-base: 200ms ease-in-out;
--transition-slow: 300ms ease-in-out;
```

### Z-Index Tokens

```css
--z-index-dropdown: 1000;
--z-index-sticky: 1020;
--z-index-fixed: 1030;
--z-index-modal-backdrop: 1040;
--z-index-modal: 1050;
--z-index-popover: 1060;
--z-index-tooltip: 1070;
```

## ASCII Wireframes

### Screen 1: Login Screen

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                                        [Support]   |
+-----------------------------------------------------------------------+
|                                                                       |
|                            Sign In                                    |
|                                                                       |
|                        +------------------+                          |
|                        |  [Login with     |                          |
|                        |   Keycloak]      |                          |
|                        +------------------+                          |
|                                                                       |
|                  Sign in to manage your tasks                       |
|                                                                       |
|                                                                       |
|                           [Terms] [Privacy]                          |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

### Screen 2: Dashboard Screen

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  [Dashboard] [Todos] [Settings] [Profile]                           |
|                                                                       |
|  Welcome back, John Doe!                                             |
|                                                                       |
|  +--------+  +--------+  +--------+  +--------+                       |
|  |  Total |  |  Done  |  |Pending |  |Deleted |                       |
|  |   42   |  |   15   |  |   27   |  |    5   |                       |
|  +--------+  +--------+  +--------+  +--------+                       |
|                                                                       |
|  [+ Create Todo]  [View All Todos]                                   |
|                                                                       |
|  Recent Todos                                                        |
|  +---------------------------------------------------------------+    |
|  | #42 Buy groceries                        [Delete]           |    |
|  |   Status: Completed                                            |    |
|  +---------------------------------------------------------------+    |
|  | #41 Call mom                             [Delete]           |    |
|  |   Status: Pending                                               |    |
|  +---------------------------------------------------------------+    |
|  | #40 Finish project report                 [Delete]           |    |
|  |   Status: Completed                                            |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

### Screen 3: Todo List Screen

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  [Dashboard] [Todos] [Settings] [Profile]                           |
|                                                                       |
|  My Todos                                                   [+ New]  |
|                                                                       |
|  [Search todos...]  [Filter: All v]  [Sort: ID v]                   |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  | #1  delectus aut autem                       [Delete]     [x]  |    |
|  |     Status: Completed                                               |    |
|  +---------------------------------------------------------------+    |
|  | #2  quis ut nam facilis et officia qui        [Delete]     [ ]  |    |
|  |     Status: Pending                                                 |    |
|  +---------------------------------------------------------------+    |
|  | #3  fugiat veniam minus                     [Delete]     [x]  |    |
|  |     Status: Completed                                            |    |
|  +---------------------------------------------------------------+    |
|  | #4  et porro tempora                       [Delete]     [ ]  |    |
|  |     Status: Pending                                                 |    |
|  +---------------------------------------------------------------+    |
|  | #5  laboriosam mollitia et enim             [Delete]     [x]  |    |
|  |     Status: Completed                                            |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  [< Prev]  Page 1 of 5  [Next >]                                      |
|  [Delete Selected] [Mark All Completed]                               |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

### Screen 4: Todo Detail Screen

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  Dashboard > Todos > Todo #42                                        |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  | #42  Buy groceries                                   [Edit]     |    |
|  |     [Completed Badge]                                             |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  User ID: 1                                                           |
|  Source: Local                                                        |
|  Created: March 9, 2026 at 10:30 AM                                  |
|  Last Modified: March 9, 2026 at 11:45 AM                            |
|                                                                       |
|  Details                                                              |
|  +---------------------------------------------------------------+    |
|  | Title: [Buy groceries                            ] [Save]       |    |
|  +---------------------------------------------------------------+    |
|  | Completed: [x] Yes                                               |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  |  [Cancel Changes]                      [Delete Todo]            |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

### Screen 5: Create/Edit Todo Screen

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  Dashboard > Todos > Create New Todo                                 |
|                                                                       |
|  [+ Create New Todo]                                                 |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  | Title *                                               |         |    |
|  | [Enter todo title...                                  ]         |    |
|  +---------------------------------------------------------------+    |
|  | Title is required                                           |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  | Completed                                                     |    |
|  | [ ] Mark as completed                                         |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  | Notes (optional)                                              |    |
|  | [Additional details or notes...                               ]         |    |
|  | [                                                             ]         |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  |  [Cancel]                           [Create Todo]             |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

### Screen 6: Settings Screen

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  [Dashboard] [Todos] [Settings] [Profile]                           |
|                                                                       |
|  Settings                                                             |
|                                                                       |
|  [Appearance] [Account] [Notifications] [Data]                       |
|                                                                       |
|  Appearance                                                           |
|  +---------------------------------------------------------------+    |
|  | Theme                                         |                 |    |
|  | [o Light] [ Dark] [ System]               |                 |    |
|  +---------------------------------------------------------------+    |
|  | Font Size                                      |       A  A  A   |    |
|  | [Small--------|-------Medium-------|--------Large]   |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  Account                                                              |
|  +---------------------------------------------------------------+    |
|  | Profile                                               |         |    |
|  | Name: John Doe                                      (read-only)   |    |
|  | Email: john@example.com                            (read-only)   |    |
|  +---------------------------------------------------------------+    |
|  | Change Password                                              |    |
|  | [Current password      ] [New password      ]                |    |
|  | [Confirm new password  ]                                   |    |
|  | [Update Password]                                            |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  Data                                                                 |
|  +---------------------------------------------------------------+    |
|  |  [Export Todos] [Import JSON] [Clear All Data]                 |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  |                               [Save Settings]                   |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

### Screen 7: Profile Screen

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  [Dashboard] [Todos] [Settings] [Profile]                           |
|                                                                       |
|  My Profile                                                           |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  |                                                               |    |
|  |                      [ Avatar Image ]                         |    |
|  |                                                               |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  Name: John Doe                                                       |
|  Username: johndoe                                                    |
|  Email: john@example.com                                              |
|                                                                       |
|  Roles                                                                |
|  +----------------+  +----------------+  +----------------+         |
|  |  todo:read     |  |  todo:write    |  |  todo:delete   |         |
|  +----------------+  +----------------+  +----------------+         |
|  +----------------+                                                    |
|  |  todo:admin    |                                                    |
|  +----------------+                                                    |
|                                                                       |
|  Session Information                                                  |
|  Logged in: March 9, 2026 at 9:00 AM                                  |
|  Token expires in: 58 minutes                                         |
|  Last activity: Just now                                             |
|                                                                       |
|  Token [Show] [Copy]                                                  |
|  +---------------------------------------------------------------+    |
|  | eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...                    [Hide] |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  |  [Refresh Token] [View Keycloak Profile]             [Logout] |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

### Screen 8: Error/Empty States

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  [Dashboard] [Todos] [Settings] [Profile]                           |
|                                                                       |
|  404 - Page Not Found                                                 |
|                                                                       |
|                          [Illustration]                               |
|                                                                       |
|                        The page you're looking                         |
|                       for doesn't exist or has                        |
|                            been moved.                                |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  |                      [Go to Dashboard]                           |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  Still having trouble? [Contact Support]                              |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  My Todos                                                   [+ New]  |
|                                                                       |
|  [Search todos...]  [Filter: All v]  [Sort: ID v]                   |
|                                                                       |
|                          [Illustration]                               |
|                                                                       |
|                        You don't have any todos                       |
|                                                                       |
|                      Create your first todo to get                    |
|                              started.                                 |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  |                     [+ Create Your First Todo]                  |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

```
+-----------------------------------------------------------------------+
|  [Logo]  Todo App                              [Avatar] [Logout]     |
+-----------------------------------------------------------------------+
|                                                                       |
|  500 - Server Error                                                  |
|                                                                       |
|                          [Illustration]                               |
|                                                                       |
|                    We encountered an unexpected error.                |
|                    Please try again later.                            |
|                                                                       |
|  Error ID: ERR-2026-03-09-001                                        |
|                                                                       |
|  +---------------------------------------------------------------+    |
|  |  [Retry]                                [Go to Dashboard]       |    |
|  +---------------------------------------------------------------+    |
|                                                                       |
|  Need help? [Contact Support]                                         |
|                                                                       |
+-----------------------------------------------------------------------+
```

## State Architecture (Zustand or equivalent)

### Store Organization

The application state is managed using Zustand with the following store slices:

#### 1. Auth Store (`useAuthStore`)
**Purpose:** Manage authentication state, user information, and JWT tokens.

**State:**
- user: object with user profile (id, username, email, name, avatar)
- token: string containing JWT access token
- tokenExpires: timestamp of token expiration
- isAuthenticated: boolean flag
- isLoading: boolean for authentication operations
- error: string for authentication error messages
- roles: array of user role strings

**Actions:**
- setAuth: function to set user and token from successful login
- clearAuth: function to clear auth state on logout
- refreshToken: function to refresh expired token with Keycloak
- checkTokenExpiry: function to verify if token is still valid
- login: async function to initiate Keycloak login flow
- logout: async function to logout and clear state

**Readers:**
- All authenticated screens (Dashboard, Todos, Create, Edit, Settings, Profile)
- Navigation component for conditional rendering
- API client for adding Authorization header

**Writers:**
- Login screen after successful Keycloak authentication
- Auth middleware on token refresh
- Logout action throughout application

**Persistence:**
- Token and user info stored in localStorage for session persistence

---

#### 2. Todos Store (`useTodosStore`)
**Purpose:** Manage todo list state, filtering, sorting, and selection.

**State:**
- todos: array of todo objects
- filteredTodos: array of todos after applying filters
- selectedTodoIds: array of selected todo IDs for batch actions
- filterStatus: string for current filter (All, Completed, Pending, Local, Remote)
- sortBy: string for current sort option (id, title, status, createdAt)
- searchTerm: string for search query
- isLoading: boolean for fetch operations
- error: string for API error messages

**Actions:**
- fetchTodos: async function to GET /api/todos
- addTodo: async function to POST /api/todos with title and completed
- updateTodo: async function to PATCH /api/todos/:id with updates
- deleteTodo: async function to DELETE /api/todos/:id
- deleteMultiple: async function to delete multiple selected todos
- toggleTodoCompletion: async function to toggle completed status
- setFilter: function to change filter status
- setSort: function to change sort option
- setSearchTerm: function to update search query
- toggleSelection: function to add/remove todo from selection
- clearSelection: function to clear all selections
- applyFiltersAndSort: function to compute filteredTodos

**Readers:**
- Todo List screen for displaying filtered and sorted list
- Dashboard screen for statistics and recent todos
- Todo Detail screen for fetching individual todo
- Create/Edit screens for adding and updating todos

**Writers:**
- Todo List screen for add, delete, filter, sort, search actions
- Todo Detail screen for update and delete actions
- Create screen for add actions
- Edit screen for update actions
- Dashboard screen for add and delete quick actions

**Persistence:**
- No persistence (in-memory store, data from backend)

---

#### 3. UI Store (`useUIStore`)
**Purpose:** Manage global UI state including theme, layout, and modal states.

**State:**
- theme: string for current theme mode (light, dark, system)
- sidebarOpen: boolean for mobile sidebar toggle
- modalOpen: string for active modal (delete-confirm, logout-confirm, etc.)
- modalData: object for modal-specific data
- notification: object for active notification (message, type, duration)
- loadingStates: object mapping screen names to loading flags
- fontScale: number for font size scaling (0.875, 1.0, 1.125)

**Actions:**
- setTheme: function to change theme mode
- toggleSidebar: function to open/close mobile sidebar
- openModal: function to open a modal with data
- closeModal: function to close active modal
- showNotification: function to display notification with auto-dismiss
- hideNotification: function to dismiss notification
- setLoading: function to set loading state for a screen
- setFontScale: function to adjust global font size

**Readers:**
- All screens for theme application
- All screens for modal and notification display
- Mobile layouts for sidebar state
- Loading indicators throughout application

**Writers:**
- Settings screen for theme and font scale changes
- All screens for modal opening/closing
- API client actions for notifications
- All screens for loading state management

**Persistence:**
- Theme and font scale stored in localStorage

---

#### 4. UserPreferences Store (`useUserPreferencesStore`)
**Purpose:** Manage user-specific settings and preferences.

**State:**
- notificationsEnabled: boolean for in-app notifications
- emailNotificationsEnabled: boolean for email notifications
- defaultFilter: string for default todo list filter
- defaultSort: string for default todo list sort
- itemsPerPage: number for pagination preference
- timezone: string for user timezone
- language: string for UI language
- lastSync: timestamp of last data sync

**Actions:**
- updatePreference: function to update a single preference
- resetPreferences: function to reset to defaults
- exportData: function to export all preferences and todos
- importData: function to import preferences and todos from file

**Readers:**
- Settings screen for displaying current preferences
- Todo List screen for applying default filter/sort/pagination
- All screens for applying language and timezone

**Writers:**
- Settings screen for updating preferences
- Import/Export actions in Settings screen

**Persistence:**
- All preferences stored in localStorage

---

### Store Interaction Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                           App Root                                   │
└─────────────────────────────────────────────────────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│ useAuthStore │          │ useUIStore   │          │ useUserPref  │
│              │          │              │          │  Store       │
│ - user       │          │ - theme      │          │              │
│ - token      │          │ - modals     │          │ - notifications│
│ - roles      │          │ - notifs     │          │ - filter     │
└──────────────┘          └──────────────┘          └──────────────┘
        │                           │                           │
        └───────────────────────────┼───────────────────────────┘
                                    │
                                    ▼
                          ┌──────────────┐
                          │ useTodosStore│
                          │              │
                          │ - todos      │
                          │ - filters    │
                          │ - selection  │
                          └──────────────┘
                                    │
                                    │ API calls with auth token
                                    ▼
                          ┌──────────────┐
                          │   Backend    │
                          │   (FastAPI)  │
                          └──────────────┘
```

### State Persistence Strategy

1. **Auth State:** Stored in `localStorage` key `auth_state`
   - Persists across page reloads
   - Cleared on explicit logout
   - Validates token expiration on app load

2. **UI Preferences:** Stored in `localStorage` key `ui_preferences`
   - Theme mode (light, dark, system)
   - Font scale
   - Sidebar state

3. **User Preferences:** Stored in `localStorage` key `user_preferences`
   - Notification settings
   - Default filter/sort
   - Pagination settings

4. **Todos State:** Not persisted (data from backend only)
   - Fetched on app load or navigation
   - Updated via API calls
   - Resets on page reload (server-side source of truth)

5. **Temporary State:** Not persisted
   - Modal states
   - Notifications
   - Loading states
   - Form input values

## WCAG 2.1 AA

### Color Contrast

**Requirements:**
- Normal text (less than 18pt): Minimum 4.5:1 contrast ratio
- Large text (18pt and above or 14pt bold): Minimum 3:1 contrast ratio
- Graphical objects and user interface components: Minimum 3:1 contrast ratio

**Implementation:**
- Primary text color (`--text-primary`) on primary background (`--background-primary`): verified 4.5:1 ratio
- Secondary text color (`--text-secondary`) on primary background: verified 4.5:1 ratio
- Link text uses `--brand-primary` color with verified 4.5:1 contrast
- Error text (`--color-error`) on error background (`--color-error-light`): verified 3:1 ratio
- Success text (`--color-success`) on success background (`--color-success-light`): verified 3:1 ratio
- Warning text (`--color-warning`) on warning background (`--color-warning-light`): verified 3:1 ratio
- Disabled button text has verified 3:1 contrast ratio
- Focus indicators use `--brand-primary` with verified 3:1 contrast
- All color combinations tested in both light and dark themes

**Contrast Testing:**
- Automated testing using axe-core or pa11y
- Manual verification of critical UI elements
- Testing across both theme modes

---

### Keyboard Accessibility

**Requirements:**
- All functionality available via keyboard
- No keyboard trap
- Logical focus order
- Visible focus indicators
- Keyboard shortcuts for common actions

**Implementation:**
- Tab navigation follows visual order: navigation, main content, actions, footer
- Enter/Space activates buttons, checkboxes, and toggles
- Escape closes modals and dropdowns
- Arrow keys navigate within lists and grids
- Home/End jumps to first/last item in lists
- Focus visible indicator (`--brand-primary` color, 2px outline)
- Skip link provided to jump to main content
- Custom keyboard shortcuts:
  - `Ctrl/Cmd + N`: Create new todo
  - `Ctrl/Cmd + F`: Focus search input
  - `Ctrl/Cmd + K`: Quick command palette
  - `Esc`: Close modal or cancel action

**Focus Management:**
- Modals trap focus within modal content
- Focus returns to triggering element on modal close
- Dynamic content updates maintain focus context
- Auto-focus on form inputs when screens load
- Descriptive focus indicators for all interactive elements

---

### Labels and Instructions

**Requirements:**
- All form fields have visible labels
- Labels programmatically associated with inputs
- Instructions and error messages are accessible
- Required fields are clearly indicated

**Implementation:**
- All form inputs have visible text labels using HTML `<label>` element
- Labels use `htmlFor` attribute to associate with inputs via `id`
- Placeholder text is used as supplement, not replacement for labels
- Required fields marked with asterisk (*) and aria-required="true"
- Error messages programmatically associated with inputs via `aria-describedby`
- Success messages provided after form submission
- Instructions for complex inputs (e.g., file upload) provided in accessible format
- Password fields have toggle visibility button with aria-label

**Aria Labels:**
- Icon-only buttons have `aria-label` describing action
- Close modal button has `aria-label="Close modal"`
- Delete buttons include `aria-label="Delete todo #id"`
- Status badges have `aria-label="Status: Completed"` or similar
- Sort/filter dropdowns have `aria-label` describing current selection

---

### Error Identification

**Requirements:**
- Errors clearly identified
- Error descriptions accessible
- Suggestions for correction provided
- Errors not disappear automatically

**Implementation:**
- Form validation errors displayed inline below field
- Error messages use descriptive text explaining the issue
- Error areas announced to screen readers via `aria-live="polite"`
- Error summaries provided for multiple validation errors
- API errors displayed in error banners with dismiss option
- Error messages include suggested corrections where applicable
- Client-side validation provides immediate feedback
- Server-side validation errors mapped to appropriate fields
- Error states use visual indicators (red border, icon)
- Error messages persist until user corrects or dismisses

**Error Types:**
- Client-side validation errors (immediate)
- Server-side validation errors (after submission)
- Network errors (connection issues)
- Authorization errors (401, 403)
- Server errors (500)
- Form submission errors
- Data loading errors

---

### No Keyboard Trap

**Requirements:**
- User can move keyboard focus away from any component
- Modals provide explicit close mechanism
- No custom focus management that prevents escape

**Implementation:**
- Tab navigation cycles through all interactive elements without getting stuck
- Modals implement focus trap that prevents leaving until closed
- Escape key closes all modals and returns focus to trigger
- Dropdowns close when focus moves outside component
- No `event.preventDefault()` on tab key except in specific cases
- Custom components implement standard focus management patterns
- Focus trap testing performed on all custom components
- Focus order follows visual DOM order

**Modal Focus Trap:**
- Focus moves to first interactive element in modal on open
- Tab cycles through modal elements only
- Escape closes modal and returns focus to trigger
- Focus returns to trigger element on modal close
- Background content has `aria-hidden="true"` when modal open

---

### Skip Link

**Requirements:**
- Skip link provided for main content
- Skip link visible on keyboard focus
- Skip link bypasses repetitive navigation

**Implementation:**
- Skip link positioned at top of page
- Skip link hidden visually (`position: absolute; left: -9999px`)
- Skip link becomes visible on focus (`left: 10px`)
- Skip link labeled "Skip to main content"
- Skip link targets `#main-content` id on main content area
- Skip link has `tabindex="0"` and is keyboard accessible
- Skip link works after page load and navigation

---

### Headings

**Requirements:**
- Headings used hierarchically
- Headings describe section content
- No skipped heading levels
- Unique headings for sections

**Implementation:**
- Page title uses `<h1>` (one per page)
- Screen sections use `<h2>` for major sections
- Subsections use `<h3>`, `<h4>` hierarchically
- No heading levels skipped (e.g., h1 followed by h3)
- Each screen has unique descriptive heading
- Headings are descriptive of content, not decorative
- Hidden headings for icon-only sections using `sr-only` class
- Modal titles use heading elements
- Breadcrumb navigation uses nav element with aria-label

**Heading Structure Example:**
```
h1: Todo App (page title, visible in browser tab)
  h2: My Todos (screen title)
    h3: Filters
    h3: Todo List
      h4: Todo Item (optional, for individual items)
  h2: Settings (screen title when navigated)
```

---

### Reduced Motion

**Requirements:**
- Respects user's reduced motion preference
- No essential motion that cannot be disabled
- Option to disable all animations

**Implementation:**
- CSS media query for `prefers-reduced-motion: reduce`
- All animations disabled when user prefers reduced motion
- Transitions disabled or instant when reduced motion active
- Keyframe animations disabled when reduced motion active
- Auto-scrolling disabled when reduced motion active
- Carousel auto-advance disabled when reduced motion active
- Loading spinners replaced with static indicators when reduced motion active
- Hover effects use only color changes, not movement
- Motion preferences stored in user settings and applied globally

**CSS Implementation:**
```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

### ARIA Attributes

**Implementation:**
- Live regions for dynamic content: `aria-live="polite"` or `aria-live="assertive"`
- Role attributes for custom components: `role="dialog"`, `role="alert"`, etc.
- `aria-expanded` for collapsible content
- `aria-selected` for tabs and listbox items
- `aria-controls` for elements controlled by other elements
- `aria-describedby` for form field error messages
- `aria-invalid="true"` for invalid form fields
- `aria-busy="true"` for loading states
- `aria-hidden="true"` for decorative elements
- `aria-label` for icon-only buttons
- `aria-current="page"` for active navigation items
- `aria-pressed` for toggle buttons
- `aria-checked` for checkboxes
- `aria-valuenow`, `aria-valuemin`, `aria-valuemax` for progress indicators

---

### Semantic HTML

**Implementation:**
- Landmark regions: header, nav, main, aside, footer
- Proper list structure: ul/ol for lists, li for list items
- Form elements: form, label, input, select, button, textarea
- Table elements: table, thead, tbody, tr, th, td (for data tables)
- Button elements for actions, not divs with click handlers
- Link elements for navigation, not divs with click handlers
- Section and article elements for content grouping
- Time and date elements for temporal information
- Figure and figcaption for images with captions

---

## White-Label Theming

### Theme Variables

The application supports white-label customization through CSS variables that can be overridden without code changes. All brand-specific elements use these variables instead of hardcoded values.

### Required Brand Variables

```css
/* Primary Brand Color */
--brand-primary: #6366f1;
--brand-primary-hover: #4f46e5;
--brand-primary-light: #e0e7ff;
--brand-primary-dark: #312e81;

/* Secondary Brand Color */
--brand-secondary: #8b5cf6;

/* Brand Logo */
--brand-logo-url: url('/logo.svg');

/* Brand Name */
--brand-name: 'Todo App';

/* Domain-specific Variables */
--brand-domain: 'example.com';
--brand-support-email: 'support@example.com';
--brand-privacy-url: '/privacy';
--brand-terms-url: '/terms';
```

### Application of Brand Variables

**Logo Display:**
- Header logo uses `--brand-logo-url` as background-image
- Logo alt text uses `--brand-name` value
- Fallback text uses `--brand-name` if logo fails to load

**Title and Headings:**
- Page title in browser tab uses `--brand-name`
- Page title H1 uses `--brand-name`
- Application name in user-facing text uses `--brand-name`

**Color System:**
- Primary buttons use `--brand-primary` background
- Primary links use `--brand-primary` text color
- Focus indicators use `--brand-primary` outline
- Active states use `--brand-primary-hover`
- Selected tabs use `--brand-primary` border-bottom

**Footer:**
- Footer copyright text includes `--brand-name`
- Support email link uses `--brand-support-email`
- Privacy and terms links use `--brand-privacy-url` and `--brand-terms-url`

**Meta Tags:**
- HTML meta title includes `--brand-name`
- OpenGraph title includes `--brand-name`

### Dynamic Theme Loading

Themes can be loaded through three methods:

1. **CSS Override (Recommended):**
   - Create `theme.css` file with variable overrides
   - Load after main stylesheets
   - Variables cascade to override defaults

2. **JavaScript Injection:**
   - Inject CSS variables via JavaScript on app load
   - Useful for dynamically loaded themes from API
   - Apply to `:root` element

3. **Configuration File:**
   - Serve `theme-config.json` from public directory
   - Application reads config and applies variables
   - Supports runtime theme switching without reload

### Theme Configuration JSON Structure

```json
{
  "brandName": "Custom Task Manager",
  "brandPrimaryColor": "#2563eb",
  "brandSecondaryColor": "#7c3aed",
  "brandLogoUrl": "/custom-logo.svg",
  "brandDomain": "custom-task.com",
  "brandSupportEmail": "help@custom-task.com",
  "brandPrivacyUrl": "/privacy-policy",
  "brandTermsUrl": "/terms-of-service"
}
```

### Default Theme (InfiniAI)

The InfiniAI theme provides the default design tokens:

**Colors:**
- Primary: #6366f1 (Indigo)
- Secondary: #8b5cf6 (Purple)
- Neutral grays for text and backgrounds
- Semantic colors for success, warning, error

**Typography:**
- System font stack for performance
- 16px base font size
- 1.5 line height for readability
- 600 font weight for headings

**Spacing:**
- 4px base unit
- 8px, 12px, 16px, 24px, 32px scale
- Consistent padding and margins

**Radius:**
- 8px base border radius
- 4px, 8px, 12px, 16px, 9999px (full) scale
- Consistent rounded corners

**Shadows:**
- 5 shadow levels from xs to xl
- Soft shadows for depth
- Consistent shadow application

### Multi-Tenant Support

For multi-tenant deployments, themes can be loaded based on subdomain or route:

```javascript
const loadTenantTheme = () => {
  const subdomain = window.location.hostname.split('.')[0];
  const themeUrl = `/themes/${subdomain}/theme.css`;

  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = themeUrl;
  document.head.appendChild(link);
};
```

### White-Label Implementation Checklist

- [ ] All brand names replaced with `--brand-name` variable
- [ ] All logos use `--brand-logo-url` variable
- [ ] All primary colors use `--brand-primary` variables
- [ ] No hardcoded product names in user-facing text
- [ ] Footer links use brand-specific URLs
- [ ] Support email configurable via variable
- [ ] Meta tags use brand name
- [ ] Page titles use brand name
- [ ] Document title uses brand name
- [ ] Error messages avoid product name references
- [ ] Loading screens use brand name
- [ ] Email templates (if any) use brand variables
- [ ] Theme testing in light and dark modes
- [ ] Theme testing across all screens

## Error and Empty States

### Per-Screen States

### Login Screen

**Loading State:**
- Spinner icon centered on screen
- Loading text: "Authenticating..."
- Form disabled during authentication
- Cancel button available to abort

**Error State:**
- Error banner at top of form
- Error message: "Authentication failed. Please try again."
- Specific error details from Keycloak (if available)
- Retry button to attempt login again
- Support link for persistent issues

**Empty State:**
- Not applicable (login form always visible)

---

### Dashboard Screen

**Loading State:**
- Skeleton loaders for statistics cards (4 cards)
- Skeleton loader for recent todos list (3 items)
- Shimmer animation effect
- Header and navigation remain visible

**Error State:**
- Error banner: "Failed to load dashboard data"
- Retry button to reload dashboard
- Specific error message from API
- Fallback: Show navigation with error message

**Empty State:**
- Illustration: Empty dashboard or no todos
- Message: "Your dashboard is ready. Start by creating your first todo."
- Call-to-action: "Create your first todo" button
- Helpful hint: "Add todos to see statistics and recent activity"

---

### Todo List Screen

**Loading State:**
- Skeleton loader for todo list (5-10 items)
- Shimmer animation
- Filters and search remain visible and functional
- Header and navigation remain visible
- Pagination skeleton at bottom

**Error State:**
- Error banner at top of list area
- Error message: "Failed to load todos. Please try again."
- Retry button with spinner
- Specific error message from API
- Fallback: Show search and filters with empty list message

**Empty State (No Todos):**
- Illustration: Empty list or clipboard
- Message: "You don't have any todos yet."
- Subtext: "Create your first todo to get started."
- Call-to-action: "+ Create your first todo" button
- Helpful hint: "Use the form above or button to add tasks"

**Empty State (No Filter Results):**
- Illustration: Search or filter empty state
- Message: "No todos match your current filters."
- Subtext: "Try adjusting your search or filter settings."
- Call-to-action: "Clear filters" button
- Helpful hint: "Your search term or filter settings may be too restrictive"

**Empty State (All Deleted):**
- Illustration: Trash or deleted items
- Message: "All your todos have been deleted."
- Subtext: "Create new todos to get back on track."
- Call-to-action: "+ Create new todo" button

---

### Todo Detail Screen

**Loading State:**
- Skeleton loader for todo details
- Shimmer animation
- Loading text: "Loading todo details..."
- Header and navigation remain visible
- Action buttons disabled

**Error State:**
- Error banner: "Failed to load todo details"
- Error message: specific error from API
- Retry button to reload todo
- Back button to return to list
- Error details for debugging (expandable)

**Empty State (Todo Not Found):**
- Illustration: 404 or not found
- Message: "Todo not found"
- Subtext: "The todo you're looking for doesn't exist or has been deleted."
- Call-to-action: "Go to todo list" button
- Helpful hint: "The todo may have been deleted by another user or session"

**Empty State (Permission Denied):**
- Illustration: Lock or unauthorized
- Message: "You don't have permission to view this todo"
- Subtext: "This todo belongs to another user or requires additional permissions."
- Call-to-action: "Go to your todos" button

---

### Create/Edit Todo Screen

**Loading State (Form):**
- Spinner on submit button
- Button text: "Creating..." or "Saving..."
- Form fields disabled during submission
- Progress indicator for long operations

**Error State (Validation):**
- Inline error messages below each field
- Field highlighted with red border
- Error message: "Title is required"
- Field marked as invalid with aria-invalid="true"
- Submit button disabled until errors resolved

**Error State (Submission):**
- Error banner at top of form
- Error message: "Failed to save todo. Please try again."
- Specific error message from API
- Retry button with spinner
- Form values preserved for retry

**Empty State (Not applicable):**
- Form always visible with pre-populated values for edit mode

---

### Settings Screen

**Loading State:**
- Skeleton loaders for settings sections
- Shimmer animation
- Header and navigation remain visible
- Save button disabled during load

**Error State:**
- Error banner: "Failed to load settings"
- Error message: specific error from API
- Retry button to reload settings
- Fallback: Show form with default values

**Empty State (Not applicable):**
- Settings always show form with current values

**Error State (Save):**
- Error banner: "Failed to save settings"
- Error message: specific error from API
- Retry button with spinner
- Values preserved for retry

---

### Profile Screen

**Loading State:**
- Skeleton loader for profile information
- Shimmer animation
- Loading text: "Loading profile..."
- Header and navigation remain visible

**Error State:**
- Error banner: "Failed to load profile"
- Error message: specific error from API
- Retry button to reload profile
- Fallback: Show logout button only

**Empty State (No Profile Data):**
- Illustration: User profile or avatar
- Message: "Profile information not available"
- Subtext: "Your profile may not be fully configured in Keycloak."
- Call-to-action: "Configure profile in Keycloak" button

**Error State (Token Refresh):**
- Error banner: "Failed to refresh token"
- Error message: specific error from Keycloak
- Retry button to refresh token
- Logout button to re-authenticate

---

### Global Error Screens

### 404 Not Found

**Display:**
- Full-screen centered layout
- Illustration: 404 or lost/missing item
- Heading: "404 - Page Not Found"
- Message: "The page you're looking for doesn't exist or has been moved."
- Action buttons: "Go to Dashboard", "Go Home"
- Support link: "Contact support"
- Navigation menu hidden

**Trigger Conditions:**
- Client-side routing to non-existent route
- Direct navigation to invalid URL
- Broken internal links

---

### 500 Server Error

**Display:**
- Full-screen centered layout
- Illustration: Server error or warning
- Heading: "500 - Server Error"
- Message: "We encountered an unexpected error. Please try again later."
- Action buttons: "Retry", "Go to Dashboard"
- Error ID: "ERR-YYYY-MM-DD-XXX" for support reference
- Support link: "Contact support"
- Navigation menu hidden

**Trigger Conditions:**
- API returns 500 status code
- Uncaught JavaScript errors
- Network failures to backend

---

### 401 Unauthorized

**Display:**
- Full-screen centered layout or modal
- Illustration: Lock or unauthorized
- Heading: "Session Expired" or "Authentication Required"
- Message: "Your session has expired. Please log in again."
- Action button: "Log in"
- Redirects to login screen after confirmation

**Trigger Conditions:**
- Token expiration detected
- API returns 401 status code
- User manually logged out

---

### 403 Forbidden

**Display:**
- Full-screen centered layout
- Illustration: Forbidden or no access
- Heading: "Access Denied"
- Message: "You don't have permission to access this resource."
- Action buttons: "Go to Dashboard", "Contact Admin"
- Support link for permission requests

**Trigger Conditions:**
- API returns 403 status code
- User lacks required role for operation
- Role-based authorization check fails

---

### Network Error

**Display:**
- Error banner at top of screen or modal
- Illustration: Network disconnected
- Heading: "Network Error"
- Message: "Unable to connect to the server. Please check your internet connection."
- Action buttons: "Retry", "Reload Page"
- Offline indicator in status bar

**Trigger Conditions:**
- Fetch API throws network error
- No internet connection
- Backend service unavailable

---

### Maintenance Mode

**Display:**
- Full-screen centered layout
- Illustration: Construction or maintenance
- Heading: "Under Maintenance"
**Message: "We're currently performing scheduled maintenance. Please check back soon."
- Estimated completion time if available
- Auto-refresh countdown
- Status page link

**Trigger Conditions:**
- API returns maintenance mode status
- Maintenance mode flag enabled in config

---

### Error Recovery Patterns

**Retry Strategy:**
- Exponential backoff for failed requests
- Maximum retry attempts: 3
- User-initiated retry via button
- Auto-retry for idempotent operations

**Fallback Content:**
- Cached data displayed if available
- Skeleton loaders during retry
- Graceful degradation for failed features

**Error Logging:**
- All errors logged to console
- Critical errors sent to error tracking service
- Error IDs generated for support reference
- User feedback mechanism for reporting issues

**User Feedback:**
- Clear, actionable error messages
- Specific guidance for resolution
- Support contact information
- Option to report issue

---

### Loading State Patterns

**Skeleton Loaders:**
- Shimmer animation effect
- Match actual content structure
- Use for lists, cards, and forms
- Disappear when data loads

**Spinner Indicators:**
- Rotating spinner icon
- Descriptive loading text
- Centered in content area
- Disabled state for forms

**Progress Bars:**
- For long-running operations (file upload, export)
- Percentage display
- Estimated time remaining
- Cancel button for interruptible operations

**Infinite Scroll Loading:**
- Spinner at bottom of list
- "Loading more items..." text
- Disabled when all items loaded

---

### State Transitions

**Loading → Success:**
- Fade in new content
- Replace skeleton with actual data
- Update loading state flags
- Dispatch success notification

**Loading → Error:**
- Display error banner
- Show specific error message
- Provide retry option
- Log error for debugging

**Error → Retry:**
- Clear error banner
- Reset loading state
- Re-initiate request
- Update UI accordingly

**Empty → Content:**
- Remove empty state illustration
- Populate with actual data
- Update state flags
- Trigger animation if applicable

# Epics and Stories (Titles + Acceptance Criteria + Technical Requirements)

Note: This breakdown is based on docs/prd-todo.md and follows the structure required by docs/templates/epic-and-stories-template.md. Epics group related functionality; each story includes concise acceptance criteria followed by technical requirements (frontend & backend) that reference the current codebase.

- Epic: Due Dates
  - Story: Add dueDate field to task model
    - Acceptance Criteria: Tasks store an optional dueDate value; format is ISO YYYY-MM-DD or absent.
    - Technical Requirements:
      - Backend: Update tasks table in packages/backend/src/app.js to include `due_date DATE` (already present in TODO schema; confirm column named due_date). Ensure INSERT/UPDATE statements accept `due_date` parameter and store NULL when absent.
      - Frontend: Update task data model (packages/frontend/src) to include `dueDate` mapping to backend `due_date` field. Use ISO YYYY-MM-DD string for transport.
  - Story: Add due date input to task creation UI
    - Acceptance Criteria: Creation form includes a due date field; entering a valid ISO date stores it on save.
    - Technical Requirements:
      - Frontend: Add a date input (HTML5 input type="date" or MUI DatePicker) in create task component; bind to task state and send to POST /api/tasks as due_date.
      - Tests: Add unit test for form to ensure date is included in payload when valid.
  - Story: Add due date edit support in task details
    - Acceptance Criteria: Users can edit and save the due date from the task details or edit view; changes persist.
    - Technical Requirements:
      - Frontend: Add editable date field in task detail/edit component; call PUT /api/tasks/:id with updated due_date.
      - Backend: Ensure PUT endpoint accepts and persists due_date; return updated task.
  - Story: Validate dueDate format and treat invalid values as absent
    - Acceptance Criteria: Non-ISO or invalid dates are ignored on save and the task stores no dueDate.
    - Technical Requirements:
      - Frontend: Validate date input client-side (ISO YYYY-MM-DD). If invalid, omit due_date key from request.
      - Backend: Sanitize/validate incoming due_date in POST/PUT; convert invalid values to NULL and do not error.

- Epic: Priority
  - Story: Add priority enum (P1, P2, P3) to task model
    - Acceptance Criteria: Tasks include a priority field limited to "P1","P2","P3" values.
    - Technical Requirements:
      - Backend: Add `priority TEXT DEFAULT 'P3'` column to tasks table (or support as TEXT column); update CREATE TABLE in packages/backend/src/app.js and ensure SELECT/INSERT/UPDATE include priority.
      - Frontend: Extend task model to include `priority` and map to backend field name.
  - Story: Default task priority to P3 when omitted
    - Acceptance Criteria: New tasks without an explicit priority are saved with priority "P3".
    - Technical Requirements:
      - Backend: Ensure DB default or server logic sets priority to 'P3' when request omits it.
      - Frontend: When creating tasks, default priority selector value to P3.
  - Story: Add priority selector to task creation/edit UI
    - Acceptance Criteria: UI provides a selector for P1/P2/P3 during create/edit and selection persists on save.
    - Technical Requirements:
      - Frontend: Add a small selector (dropdown or radio) in create/edit components; include selected priority in API payload.
      - Tests: Add UI test verifying selected value is sent to backend.

- Epic: Filters & Views
  - Story: Implement All, Today, and Overdue tabs
    - Acceptance Criteria: Tabs for All, Today, Overdue are present and switch views when clicked.
    - Technical Requirements:
      - Frontend: Add tabs in main task list component (packages/frontend/src/components). Implement route or state to switch view.
      - Backend: Support query parameters on GET /api/tasks (e.g., completed, due_date filters). Existing GET supports completed and search; extend to accept date range or client-side filter.
  - Story: Show completed tasks in All view
    - Acceptance Criteria: All view displays both completed and incomplete tasks.
    - Technical Requirements:
      - Frontend: For All tab, request GET /api/tasks without completed filter or explicitly completed=all; render completed tasks with completed styling.
      - Backend: GET /api/tasks already supports completed query param; ensure omitted behavior returns both.
  - Story: Hide completed tasks in Today and Overdue views
    - Acceptance Criteria: Today and Overdue views only show tasks where completed=false.
    - Technical Requirements:
      - Frontend: For Today/Overdue tabs, request GET /api/tasks?completed=false and filter by due_date on client or via backend query params.
      - Backend: Support completed=false filter server-side (already implemented).
  - Story: Filter tasks for Today view by due date
    - Acceptance Criteria: Today view shows incomplete tasks with dueDate equal to local current date.
    - Technical Requirements:
      - Frontend: Compute local current date (YYYY-MM-DD) and call GET /api/tasks?completed=false&date=YYYY-MM-DD or fetch all and filter client-side.
      - Backend: Optionally implement date query param handling in GET /api/tasks to filter by due_date = provided date.
  - Story: Filter tasks for Overdue view by due date
    - Acceptance Criteria: Overdue view shows incomplete tasks with dueDate before local current date.
    - Technical Requirements:
      - Backend: Add support to filter due_date < current date when requested, or accept a `before` query param.
      - Frontend: Prefer server filtering via GET /api/tasks?completed=false&due_before=YYYY-MM-DD for efficiency.

- Epic: Local Storage & Validation
  - Story: Persist tasks and new fields to local storage
    - Acceptance Criteria: Tasks, dueDate, and priority persist across browser reloads using local storage only.
    - Technical Requirements:
      - Note: Repo currently contains a Node backend and React frontend with proxy. Decide an implementation approach:
        - Option A (recommended given repo): Persist via backend API (packages/backend) using SQLite in-memory for dev; consider switching to file-backed DB for persistence in future.
        - Option B (if strictly following PRD local-only): Implement client-side localStorage persistence in packages/frontend/src and avoid backend calls; update UI to read/write localStorage keys and tests.
      - Frontend (localStorage option): Implement serialization/deserialization helpers and migration strategy for date/priorities.
  - Story: Enforce required title validation on create/edit
    - Acceptance Criteria: Create/edit form prevents save and shows validation when title is empty.
    - Technical Requirements:
      - Frontend: Add client-side validation to form components; disable save when title blank and show inline error.
      - Backend: POST/PUT endpoints already validate title and return 400; maintain parity with frontend validation.

- Epic: Post‑MVP Overdue Highlighting
  - Story: Add visual highlight for overdue tasks
    - Acceptance Criteria: Overdue tasks are visually distinct (e.g., red text or badge) in task list and task details in Post‑MVP release.
    - Technical Requirements:
      - Frontend: Add visual CSS class or MUI variant for overdue (detected by comparing dueDate < local date). Use consistent variables in components.

- Epic: Post‑MVP Sorting
  - Story: Sort tasks with overdue first
    - Acceptance Criteria: Task list ordering places overdue tasks before others when sorting is enabled.
    - Technical Requirements:
      - Backend: Update GET /api/tasks ORDER BY clause to support sorting: overdue first (CASE WHEN due_date < CURRENT_DATE THEN 0 ELSE 1 END), then priority, then due_date ASC, then undated last. Current backend ORDER BY due_date IS NULL, due_date ASC, created_at ASC should be extended.
      - Frontend: If server-side sorting not used, implement client-side comparator implementing the same rules.
  - Story: Sort tasks by priority (P1→P3)
    - Acceptance Criteria: Within non-overdue groups, tasks are ordered by priority descending P1 then P2 then P3.
    - Technical Requirements:
      - Backend: Add priority sorting to ORDER BY clause (map P1->1, P2->2, P3->3 or use CASE expression).
  - Story: Sort tasks by due date ascending
    - Acceptance Criteria: Within same priority, tasks are ordered by dueDate ascending; undated tasks are last.
    - Technical Requirements:
      - Backend/Frontend: Ensure comparator puts NULL due dates last (e.g., ORDER BY due_date IS NULL, due_date ASC).
  - Story: Ensure undated tasks appear last
    - Acceptance Criteria: Tasks without dueDate are always listed after dated tasks in sorted views.
    - Technical Requirements:
      - Backend: Use due_date IS NULL in ORDER BY as already present; verify when combined with other sorts.

- Epic: Post‑MVP UX Polish
  - Story: Add color-coded priority badges (P1 red, P2 orange, P3 gray)
    - Acceptance Criteria: Each task displays a badge colored per priority mapping (P1 red, P2 orange, P3 gray).
    - Technical Requirements:
      - Frontend: Implement Badge component or use MUI Chip with color mapping; ensure accessible color contrast.
  - Story: Polish task list UI and small visual refinements
    - Acceptance Criteria: Minor UI improvements applied (spacing, badge alignment, readable colors); no functional changes.
    - Technical Requirements:
      - Frontend: Update CSS/MUI theme variables in packages/frontend/src/theme or component styles.

Notes on Implementation Choices & References to Codebase

- Repo structure: Monorepo with React frontend at packages/frontend and Node/Express backend at packages/backend. package.json at repo root uses npm workspaces and proxy configured for frontend to http://localhost:3030.
- Backend reference: packages/backend/src/app.js contains existing tasks schema and endpoints (GET/POST/PUT/PATCH/DELETE). It already defines `due_date` column and basic validation for `title` — extend this file to add `priority` and query param handling for due date filters and sorting as described.
- Frontend reference: packages/frontend uses React and MUI (package.json). Update components under packages/frontend/src (task list, task form, task detail) to add inputs, tabs, axios calls to backend endpoints, and client-side validation.
- Testing: Add unit tests for frontend components (React Testing Library) and integration tests for backend endpoints (supertest or similar). Existing test scripts are present in package.json workspaces.


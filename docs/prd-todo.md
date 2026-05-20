# Product Requirements Document (PRD) - TODO App: Due Dates, Priority & Filters

## 1. Overview

We are upgrading the basic TODO app to help users organize tasks by adding optional due dates, priority levels, and quick filters. The goal is a small, teachable MVP that improves task visibility while keeping storage local and the UI simple.

---

## 2. MVP Scope

- Add optional `dueDate` field (ISO YYYY-MM-DD). Invalid dates are ignored/treated as absent.
- Add `priority` enum: `P1 | P2 | P3` with default `P3`.
- Filters/Tabs: All, Today, Overdue. "Today" and "Overdue" views show only incomplete tasks; "All" shows completed tasks too.
- Data validation: `title` required; `priority` defaults to `P3` when omitted; `dueDate` optional.
- Local-only storage (no backend/external storage changes).
- Simple UI additions: ability to set/edit due date and priority when creating/editing a task.

---

## 3. Post-MVP Scope

- Visual overdue highlighting (e.g., red background or badge for overdue tasks).
- Sorting: overdue first → priority (P1→P3) → due date ascending → undated tasks last.
- Color-coded priority badges (P1 red, P2 orange, P3 gray) and other small UX polish.

---

## 4. Out of Scope

- Notifications and reminders.
- Recurring tasks.
- Multi-user or shared storage features.
- Keyboard navigation and advanced accessibility features (beyond baseline).
- External/backend storage; system remains local-only for MVP.

---


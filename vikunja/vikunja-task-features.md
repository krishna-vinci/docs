# Vikunja Task Features

This document provides an overview of the features available within a task in Vikunja, covering both the frontend experience and the underlying backend/database flow.

## 1. Frontend Features (Task Detail View)

When a task is opened in the frontend, the following features are available:

### Core Attributes
*   **Title:** Editable heading of the task.
*   **Description:** A rich-text area (using TipTap) that supports Markdown, formatting, and file uploads.
*   **Status Toggle:** A toggle to mark the task as **Done** or **Undone**.
*   **Favorite:** A "Star" icon to add the task to the "Important" pseudo-project.
*   **Subscription:** A toggle to follow/unfollow the task for notifications.

### Organization & Metadata
*   **Labels:** Tags to categorize tasks across projects.
*   **Priority:** Set a priority level (ranging from "Low" to "Urgent").
*   **Color:** Assign a specific hex color to the task card.
*   **Assignees:** Search and assign users or teams to the task.
*   **Progress:** A percentage slider (0–100%) to track completion progress.

### Dates & Reminders
*   **Due Date:** When the task is expected to be finished.
*   **Start Date:** When work on the task should begin.
*   **End Date:** Used for Gantt charts and specific duration tracking.
*   **Reminders:** Multiple alerts (e.g., relative to due date or a specific absolute date).
*   **Repeat:** Set the task to recur automatically (daily, weekly, monthly, or custom).

### Collaboration & Content
*   **Comments:** A threaded discussion section for team communication.
*   **Attachments:** Upload files, images, or documents directly to the task.
*   **Reactions:** Add emojis to the task (similar to GitHub or Slack).

### Relationships & Management
*   **Related Tasks:** Link tasks with relationships like "is blocked by," "belongs to," or "is subtask of."
*   **Move Project:** Move the task to a different project.
*   **Duplicate:** Create an exact copy of the task.
*   **Delete:** Permanently remove the task.

---

## 2. Backend & Database Flow

### General Architecture
Vikunja's backend is written in Go and uses the XORM ORM for database interactions.

### Data Flow
1.  **Frontend Action:** User modifies a field (e.g., changes priority).
2.  **API Request:** The frontend sends a `POST` request to `/api/v1/tasks/:id` with the updated JSON.
3.  **Route Handling:** The request is caught by the routes defined in `pkg/routes/routes.go` and handled by the generic web handler.
4.  **Model Logic:** The `Task.Update()` method in `pkg/models/tasks.go` is executed.
    *   It first validates the user's permissions.
    *   It updates standard columns in the `tasks` table.
    *   For relational data (reminders, assignees), it triggers specific sub-methods like `updateReminders()`.
5.  **Database Storage:** XORM persists the changes to the database.
6.  **Events & Notifications:** A `TaskUpdatedEvent` is fired, which may trigger notifications (emails, push) via listeners in `pkg/models/listeners.go`.

### Database Schema Mapping
| Feature | Table(s) | Key Columns |
| :--- | :--- | :--- |
| **Basic Info** | `tasks` | `id`, `title`, `description`, `priority`, `done`, `hex_color` |
| **Reminders** | `task_reminders` | `task_id`, `reminder`, `relative_period`, `relative_to` |
| **Labels** | `label_tasks`, `labels` | `task_id`, `label_id` |
| **Assignees** | `task_assignees` | `task_id`, `user_id` |
| **Attachments** | `task_attachments` | `task_id`, `file_id`, `name` |
| **Comments** | `task_comments` | `task_id`, `content`, `author_id` |
| **Relations** | `task_relations` | `task_id`, `other_task_id`, `relation_kind` |

# User Requirements Specification — Todo App

| Field | Value |
|---|---|
| Document ID | TODO-URS-001 |
| Version | 1.0 |
| Status | Draft |
| Author | Platform Team |
| References | TODO-BRD-001 |

---

## 1. User Personas

### 1.1 Developer Dana

- **Role:** Senior Software Engineer
- **Goals:** Quickly capture tasks without leaving the terminal, track blockers, see what teammates are working on
- **Frustrations:** Context-switching between tools, duplicate status updates in Slack and spreadsheets
- **Tech comfort:** High — comfortable with CLI and keyboard shortcuts

### 1.2 Product Manager Pat

- **Role:** Product Manager
- **Goals:** Maintain a prioritised backlog, assign tasks, track delivery progress across teams
- **Frustrations:** Lack of cross-team visibility, no single source of truth for sprint status
- **Tech comfort:** Medium — prefers GUI, not comfortable with APIs

### 1.3 Operations Lead Alex

- **Role:** Operations Lead
- **Goals:** Track operational tasks, assign on-call actions, maintain runbooks as tasks
- **Frustrations:** Tasks getting lost in Slack threads, no audit trail for completed work
- **Tech comfort:** Medium

---

## 2. Functional Requirements

### 2.1 Authentication

| ID | Requirement | Priority |
|---|---|---|
| FR-AUTH-01 | Users must be able to sign in using their GitHub account via OAuth | Must |
| FR-AUTH-02 | Users must remain authenticated across browser sessions until they explicitly sign out | Must |
| FR-AUTH-03 | Unauthenticated users must see a sign-in prompt and cannot access any task data | Must |

### 2.2 Task Management

| ID | Requirement | Priority |
|---|---|---|
| FR-TASK-01 | Users must be able to create a task with a title | Must |
| FR-TASK-02 | Users must be able to add an optional description to a task (markdown supported) | Must |
| FR-TASK-03 | Users must be able to assign a task to any workspace member | Must |
| FR-TASK-04 | Users must be able to set a due date on a task | Must |
| FR-TASK-05 | Users must be able to set task priority: Critical, High, Medium, Low | Must |
| FR-TASK-06 | Users must be able to mark a task as complete | Must |
| FR-TASK-07 | Users must be able to reopen a completed task | Must |
| FR-TASK-08 | Users must be able to delete a task they created | Must |
| FR-TASK-09 | Users must be able to add comments to a task | Should |
| FR-TASK-10 | Users must be able to attach labels to tasks | Should |
| FR-TASK-11 | Users must be able to set task dependencies (blocks / blocked-by) | Could |

### 2.3 Organisation

| ID | Requirement | Priority |
|---|---|---|
| FR-ORG-01 | Users must be able to create projects to group related tasks | Must |
| FR-ORG-02 | Users must be able to view tasks filtered by project | Must |
| FR-ORG-03 | Users must be able to view tasks assigned to them | Must |
| FR-ORG-04 | Users must be able to view tasks by due date | Must |
| FR-ORG-05 | Users must be able to search tasks by title | Should |
| FR-ORG-06 | Users must be able to create custom labels with colours | Should |

### 2.4 Notifications

| ID | Requirement | Priority |
|---|---|---|
| FR-NOTIF-01 | Users must receive a notification when a task is assigned to them | Must |
| FR-NOTIF-02 | Users must receive a notification when a task they created is commented on | Should |
| FR-NOTIF-03 | Users must receive a due-date reminder 24 hours before a task is due | Should |

---

## 3. Non-Functional Requirements

### 3.1 Performance

| ID | Requirement |
|---|---|
| NFR-PERF-01 | Task list must render within 1 second on a standard broadband connection |
| NFR-PERF-02 | Task creation must complete (round-trip) within 2 seconds |
| NFR-PERF-03 | Search results must appear within 500ms of input |

### 3.2 Usability

| ID | Requirement |
|---|---|
| NFR-USE-01 | The interface must be operable entirely via keyboard |
| NFR-USE-02 | The interface must be responsive and usable on screens ≥ 375px wide |
| NFR-USE-03 | All interactive elements must have visible focus indicators |
| NFR-USE-04 | WCAG 2.1 AA compliance required |

### 3.3 Security

| ID | Requirement |
|---|---|
| NFR-SEC-01 | All API communication must be over HTTPS |
| NFR-SEC-02 | OAuth tokens must never be logged or exposed in error messages |
| NFR-SEC-03 | Users must only be able to access tasks within workspaces they are members of |

### 3.4 Reliability

| ID | Requirement |
|---|---|
| NFR-REL-01 | Target availability: 99.5% (excluding GitHub platform outages) |
| NFR-REL-02 | Data must not be lost if the user's browser crashes mid-edit |

---

## 4. Use Cases

### UC-01: Create a Task

**Actor:** Any authenticated user
**Precondition:** User is signed in and has selected a project
**Main Flow:**
1. User clicks "New task"
2. User enters a title
3. User optionally sets description, assignee, priority, and due date
4. User clicks "Create"
5. Task appears at the top of the project task list

**Alternate Flow A — Missing Title:**
- Step 2a: User submits without a title → system shows inline validation error, task is not created

### UC-02: Complete a Task

**Actor:** Any authenticated user
**Precondition:** Task exists and is open
**Main Flow:**
1. User clicks the completion checkbox on a task
2. Task is marked complete and moves to the "Completed" section
3. Assignee and creator receive a completion notification

### UC-03: Filter Tasks by Assignee

**Actor:** Project Manager Pat
**Main Flow:**
1. User opens a project view
2. User selects "Assigned to me" from the filter dropdown
3. Task list updates to show only tasks assigned to the current user

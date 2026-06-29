# API Specification — Todo App

| Field | Value |
|---|---|
| Document ID | TODO-API-001 |
| Version | 1.0 |
| Status | Draft |
| Base | GitHub REST API v3 (`https://api.github.com`) |

> The Todo App does not expose its own API — it is a client of the GitHub REST and GraphQL APIs. This document specifies which GitHub endpoints are used, with what parameters, and under what conditions.

---

## 1. Authentication

All requests include:
```
Authorization: Bearer {github_oauth_token}
Accept: application/vnd.github+json
X-GitHub-Api-Version: 2022-11-28
```

---

## 2. Endpoints Used

### 2.1 User

#### Get authenticated user
```
GET /user
```
Called on app load to verify token and populate user profile.

**Response fields used:** `login`, `name`, `avatar_url`, `html_url`

---

### 2.2 Projects (Milestones)

#### List projects
```
GET /repos/{owner}/{repo}/milestones
  ?state=open
  &per_page=50
  &sort=due_on
  &direction=asc
```

#### Create project
```
POST /repos/{owner}/{repo}/milestones
Body: { "title": string, "description": string, "due_on": string | null }
```

#### Update project
```
PATCH /repos/{owner}/{repo}/milestones/{milestone_number}
Body: { "title"?: string, "description"?: string, "due_on"?: string, "state"?: "open"|"closed" }
```

---

### 2.3 Tasks (Issues)

#### List tasks
```
GET /repos/{owner}/{repo}/issues
  ?milestone={number|"*"|"none"}
  &assignee={login|"*"|"none"}
  &labels={comma-separated label names}
  &state={open|closed|all}
  &sort={created|updated|comments}
  &direction={asc|desc}
  &per_page=50
  &page={n}
```

#### Get single task
```
GET /repos/{owner}/{repo}/issues/{issue_number}
```

#### Create task
```
POST /repos/{owner}/{repo}/issues
Body: {
  "title": string,
  "body": string,
  "assignees": string[],
  "labels": string[],
  "milestone": number | null
}
```

#### Update task
```
PATCH /repos/{owner}/{repo}/issues/{issue_number}
Body: {
  "title"?: string,
  "body"?: string,
  "assignees"?: string[],
  "labels"?: string[],
  "milestone"?: number | null,
  "state"?: "open" | "closed"
}
```

---

### 2.4 Comments

#### List comments on a task
```
GET /repos/{owner}/{repo}/issues/{issue_number}/comments
  ?per_page=50
```

#### Add comment
```
POST /repos/{owner}/{repo}/issues/{issue_number}/comments
Body: { "body": string }
```

#### Edit comment
```
PATCH /repos/{owner}/{repo}/issues/comments/{comment_id}
Body: { "body": string }
```

#### Delete comment
```
DELETE /repos/{owner}/{repo}/issues/comments/{comment_id}
```

---

### 2.5 Labels

#### List labels
```
GET /repos/{owner}/{repo}/labels
  ?per_page=100
```

#### Create label
```
POST /repos/{owner}/{repo}/labels
Body: { "name": string, "color": string, "description": string }
```

---

### 2.6 GraphQL — Activity Timeline

Used to fetch rich activity (assignments, label changes, state changes) alongside comments in a single request:

```graphql
query TaskActivity($owner: String!, $repo: String!, $number: Int!) {
  repository(owner: $owner, name: $repo) {
    issue(number: $number) {
      timelineItems(first: 100, itemTypes: [
        ISSUE_COMMENT,
        ASSIGNED_EVENT,
        UNASSIGNED_EVENT,
        LABELED_EVENT,
        UNLABELED_EVENT,
        CLOSED_EVENT,
        REOPENED_EVENT,
        RENAMED_TITLE_EVENT
      ]) {
        nodes {
          __typename
          ... on IssueComment {
            id
            body
            createdAt
            author { login avatarUrl }
          }
          ... on AssignedEvent {
            createdAt
            assignee { ... on User { login } }
            actor { login }
          }
          ... on LabeledEvent {
            createdAt
            label { name color }
            actor { login }
          }
          ... on ClosedEvent {
            createdAt
            actor { login }
          }
        }
      }
    }
  }
}
```

---

## 3. Rate Limiting

GitHub enforces 5,000 requests per hour per authenticated token (REST) and 500,000 points per hour (GraphQL). The app implements:

- **Request deduplication** via TanStack Query's default behaviour
- **Stale-while-revalidate** to reduce redundant fetches
- **429 handling** — back-off using `X-RateLimit-Reset` header timestamp
- **Rate limit display** — remaining quota shown in dev mode footer

# ADR-001: Use GitHub Issues as the Task Data Store

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-06-28 |
| Deciders | Platform Team |

---

## Context

We need a persistent data store for tasks (title, description, assignee, due date, priority, comments, activity). Options considered:

1. **Custom REST API + database** (e.g. Postgres + Express)
2. **Firebase / Firestore**
3. **GitHub Issues** (native GitHub primitive)
4. **GitHub Projects v2 (GraphQL)**

The system must have zero external infrastructure cost and must keep data co-located with the codebase.

---

## Decision

Use **GitHub Issues** as the sole data store for tasks.

- Tasks = Issues
- Projects = Milestones
- Labels = Label with naming conventions (`priority:*`, `due:*`)
- Comments = Issue Comments
- Activity = Issue Timeline Events (GraphQL)

---

## Consequences

**Positive:**
- Zero database infrastructure to operate or pay for
- Data lives alongside code in the same repository
- Inherits GitHub's authentication, access control, and audit log
- GitHub Issues UI provides a fallback interface if the SPA is unavailable
- Full GitHub API ecosystem (webhooks, Actions triggers, integrations)

**Negative:**
- GitHub API rate limits (5,000 req/hr) constrain scale for large teams
- No custom query capabilities beyond what the GitHub API exposes
- Label namespace is shared with regular repo labels — naming conventions must be enforced
- Cannot store arbitrary structured metadata without encoding it in the issue body

**Mitigated by:**
- TanStack Query caching reduces API calls significantly
- Label conventions documented and enforced via CI label check

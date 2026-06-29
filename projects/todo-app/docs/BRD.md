# Business Requirements Document — Todo App

| Field | Value |
|---|---|
| Document ID | TODO-BRD-001 |
| Version | 1.0 |
| Status | Draft |
| Author | Product Team |
| Date | 2026-06-28 |

---

## 1. Executive Summary

The organisation currently has no standardised task management solution. Engineering, product, and operations teams each use disparate tools (spreadsheets, sticky notes, ad-hoc Slack threads) to track work items. This creates visibility gaps, duplicated effort, and missed deadlines. The Todo App will provide a unified, lightweight task management platform that integrates with existing GitHub workflows and requires zero infrastructure overhead for teams already operating inside GitHub.

---

## 2. Business Context

### 2.1 Problem Statement

Teams report an average of 2.4 hours per week lost to task coordination overhead — locating task status, chasing updates, and reconciling work across tools. Engineering sprints regularly carry over 30–40% of planned items due to poor visibility into cross-team dependencies.

### 2.2 Strategic Alignment

This initiative supports the organisation's FY26 goal of reducing operational friction by 20%. A lightweight, developer-friendly task tool reduces context-switching and keeps work close to where it is executed (code repositories).

### 2.3 Opportunity

| Metric | Current State | Target State |
|---|---|---|
| Task coordination overhead | 2.4 hrs/week/person | < 0.5 hrs/week/person |
| Sprint carryover rate | 35% | < 15% |
| Cross-team dependency visibility | Manual (ad hoc) | Automated via labels/assignments |

---

## 3. Stakeholders

| Role | Name / Team | Interest |
|---|---|---|
| Sponsor | VP Engineering | Cost reduction, delivery predictability |
| Product Owner | Platform Team | Feature scope, prioritisation |
| Primary Users | Engineering, Product, Ops | Daily task management |
| Secondary Users | Leadership | Reporting, visibility |
| IT / Security | Infra Team | Data residency, access control |

---

## 4. Business Objectives

1. **BO-01** — Reduce task coordination overhead by 80% within 90 days of launch
2. **BO-02** — Achieve 70% weekly active usage across target teams within 60 days
3. **BO-03** — Eliminate dependency on external paid task tools, saving ≥ $12k/year
4. **BO-04** — Enable cross-team task visibility without requiring manual status updates

---

## 5. Scope

### 5.1 In Scope

- Task creation, assignment, and prioritisation
- Project and label-based organisation
- Due date tracking with reminders
- Activity history per task
- GitHub authentication (no separate user management)
- Mobile-responsive web interface

### 5.2 Out of Scope

- Native mobile applications (iOS / Android)
- Time tracking / billing integration
- Resource capacity planning
- External integrations (Jira, Linear, Asana) in v1

### 5.3 Assumptions

- All users have existing GitHub accounts
- Teams operate primarily on desktop browsers
- No offline / disconnected usage requirement in v1

---

## 6. Business Constraints

| Constraint | Detail |
|---|---|
| Budget | Development only — no SaaS tooling costs |
| Timeline | MVP in 8 weeks |
| Data residency | All data must remain within GitHub (no third-party storage) |
| Authentication | GitHub OAuth only — no separate credential store |

---

## 7. Success Criteria

- Weekly active users ≥ 70% of enrolled team members after 60 days
- Task coordination Slack messages reduced by ≥ 50% (measured via Slack analytics)
- Zero P1 security incidents in first 6 months
- NPS from pilot teams ≥ 40 after 30 days

---

## 8. Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Low adoption due to habit inertia | High | High | Phased rollout, champions in each team |
| GitHub API rate limits under load | Medium | Medium | Implement caching layer, monitor usage |
| Scope creep from feature requests | High | Medium | Strict v1 scope gate, public backlog |
| GitHub outage affecting availability | Low | High | Graceful degradation, offline read mode |

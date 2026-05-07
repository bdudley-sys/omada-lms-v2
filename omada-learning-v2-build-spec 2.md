# Omada Learning v2 — Build Handoff Spec

**Author:** Bruce Dudley  
**Prepared for:** LMS Space build agent and reviewers  
**Status:** Ready for build handoff  
**Date:** 2026-05-06

## Purpose

This document is the clean handoff brief for building Omada Learning v2 inside the LMS Space. It compresses the approved product spec into an agent-ready implementation brief. The source Omada Learning v2 spec remains the binding source of truth; this handoff exists to make the next action the build, not another planning loop.

## Build objective

Build Omada Learning v2 inside the LMS Space, preserving the v1 system that already works and adding two major layers:

- Best-in-class LMS nomenclature aligned to standard field language.
- A visibility layer for managers and administrators built around Development Plans and Operations.

The build must preserve the working parts of v1:

- Tracked / Recommended / Open tier model.
- Reviewed-exercise loop.
- Manager review queue.

## Binding rules

- The supplied Omada Learning v2 spec is the source of truth.
- If implementation pressure conflicts with the spec, the spec wins unless amended explicitly.
- Do not add features outside the written scope.
- Do not collapse learner, manager, and admin surfaces into a shared dashboard model.
- Keep mastery and completion distinct.
- Keep instrumentation ahead of automation.

## Product boundaries

Build only what is inside scope.

### In scope

- Terminology pass and lifecycle normalization.
- Competency model.
- Mastery model.
- Manager-owned Development Plans.
- Team Development roster view.
- Learner read-only Development Plan mirror.
- Admin Operations visibility layer.
- Signals, audit visibility, paths, and cohorts as sequenced in the build plan.

### Explicitly out of scope

- AI-generated coaching nudges.
- Automated content tagging.
- xAPI / SCORM compliance.
- Public certificates or badging.
- E-commerce or paid courses.
- External LXP federation.
- Predictive analytics.
- Mobile native apps.

## Audience model

Omada Learning v2 has three audiences. Each gets the view it needs and no more.

### Learner surface

- For Today.
- Coming Up.
- My Paths.
- Course Catalog.
- My Profile.
- My Development Plan.

### Manager surface

- Manager Dashboard.
- Review Queue.
- Review Detail.
- Team Development.
- Development Plan.

### Admin surface

- Admin Overview.
- Users.
- Courses.
- Compliance.
- Operations.
- Competency Library.
- Audit Log.
- Cohorts later: optional in v2.0, required in v2.1.

## Terminology contract

Adopt these UI terms:

| Current / informal | v2 UI term |
|---|---|
| User | Person |
| Item | Activity |
| `kind` | Activity Type |
| lesson / quiz / reviewed_exercise | Lesson / Quiz / Reviewed Exercise |
| informal path language | Path |
| none | Cohort |
| informal rule assignment | Audience |

### Retired learner-facing labels

- "submitted to your manager" → **Sent for review**
- "needs follow-up" → **Action needed**
- "coach" as a visible status label → **Coaching note**

## Lifecycle contract

Keep three separate lifecycles.

### Enrollment lifecycle

- `not_enrolled`
- `enrolled`
- `in_progress`
- `completed`
- `waived`
- `expired`

### Activity lifecycle

- `locked`
- `available`
- `in_progress`
- `submitted`
- `under_review`
- `passed`
- `redo_requested`

### Submission lifecycle

- `draft`
- `submitted`
- `under_review`
- `pass`
- `redo`
- `coach`

### Hard rule

For reviewed exercises, Activity status is derived from Submission status. Do not create two sources of truth.

## New domain objects

Add the following objects to the data model:

- Competency
- ActivityCompetency
- Gap
- Mastery
- DevelopmentPlan
- Checkpoint
- ManagerNote
- Path
- CohortRun
- Signal

## Existing model extensions

Extend current entities as follows:

- `Item` gets denormalized `competencyIds` for read-side use.
- `Course` gets `pathIds`.
- `User` gets `audienceTags`.

Truth remains in normalized mapping tables, not denormalized arrays.

## Development Plan page contract

The manager-owned Development Plan page answers one question:

**Where is this person, and what am I doing about it?**

### Required page structure

1. **Where they are** — course-by-course progress.
2. **Where the lift is** — competency gaps with severity and rationale.
3. **What I'm signing off on, when** — checkpoint timeline.
4. **Manager notes** — append-only private manager log with 24-hour edit grace.

### Allowed actions

- Add, edit, or remove checkpoint.
- Sign off checkpoint.
- Edit gap severity with reason.
- Add manager note.
- Open linked course or activity.

### Not allowed

- No second submission queue.
- No goal-setting system.
- No learner edit access.
- No learner access to private manager notes.

## Learner Development Plan mirror

Build a read-only learner mirror of the Development Plan.

### Learner can see

- Assigned paths.
- Named competency gaps.
- Upcoming checkpoints.
- Visible manager comments intended for learner view.

### Learner cannot see

- Private manager notes.
- Any gap where `severity = high`.

The `high` severity filter must be enforced server-side.

## Operations page contract

Build the following seven Operations widgets:

1. Bottlenecks board.
2. Review SLA panel.
3. Mastery vs. completion divergence.
4. Audience drift.
5. Live activity feed.
6. Course adoption curve.
7. At-risk strip.

### Not in v2.0 Operations

- Predictive forecasting.
- Automated nudges.
- Cross-system HRIS or performance-system correlation.

## Binding resolved decisions

These are implementation constraints, not open questions.

1. Competency editing is admin-only.
2. Learners never see `high` severity gaps.
3. Mastery is monotonic and does not decay.
4. Development Plans may exist without a Path or Cohort.
5. Single sign-off model only.
6. Notifications are in-app only in v2.

## API surface

Add route groups for:

- `/api/admin/competencies`
- `/api/manager/...development-plans`
- `/api/manager/...checkpoint`
- `/api/learner/:learnerId/gaps`
- `/api/learner/:learnerId/mastery`
- `/api/admin/operations...`
- `/api/admin/paths`
- `/api/admin/cohort-runs`

Rules:

- Use REST verbs.
- Use resource-shaped paths.
- Manual manager gap overrides beat derived gaps.
- Admin recompute actions must remain auditable.

## Build sequence

Build in this order and treat each sprint as a working checkpoint.

### Sprint 0

- Terminology pass only.
- Normalize lifecycle labels.
- No schema changes.

### Sprint 1

- Add Competency, ActivityCompetency, Mastery.
- Build Competency Library.
- Add activity-to-competency mapping UI.
- Implement mastery recompute.

### Sprint 2

- Add DevelopmentPlan, Checkpoint, ManagerNote, Gap.
- Build Team Development.
- Build manager Development Plan.
- Build learner Development Plan mirror.

### Sprint 3

- Add Signal table.
- Add Operations derivations and page.
- Add in-app manager Signals.
- Promote Audit Log to its own route.

### Sprint 4

- Add Path and CohortRun.
- Build Paths page.
- Build Cohorts page.
- Tie audience rules to enrollment.

## Agent execution prompt

Use the following prompt as the direct build brief:

> Build Omada Learning v2 inside the LMS Space from the attached spec.  
> Treat the spec as binding.  
> Build in Sprint 0 through Sprint 4 order exactly.  
> Do not add features outside the spec.  
> Preserve the three-surface model: Learner, Manager, Admin.  
> Keep mastery and completion distinct.  
> Keep lifecycle models separated into enrollment, activity, and submission.  
> For reviewed exercises, derive activity state from submission state.  
> Enforce learner privacy rules around high-severity gaps server-side.  
> Use the resolved decisions in §10 as hard constraints.  
> Stop only for true ambiguity or contradiction.  
> First output: implementation backlog, schema delta plan, API contract set, and page/component contract set.  
> Second output: start Sprint 0 build.

## Immediate next action

The next move is not more planning. The next move is:

1. Extract the implementation backlog.
2. Draft schema deltas.
3. Draft API contracts.
4. Start Sprint 0.

That is the handoff point.

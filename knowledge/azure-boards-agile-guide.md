---
name: azure-boards-agile-guide
description: Contains official Azure Boards Agile guidance, including definitions, field rules, and best practices for writing User Stories, Acceptance Criteria, and Tasks.
---

# Azure Boards Agile Work Item & Best Practices Guide

## 1. Hierarchy & Work Item Types (WITs)
In Azure Boards (Agile Process), work items follow a strict hierarchy:
- **Epic:** Represents high-level business initiatives spanning multiple releases.
- **Feature:** Represents customer-facing capabilities that deliver distinct value.
- **User Story:** Represents specific end-user requirements scoped to a single sprint/iteration.
- **Task:** Represents granular technical implementation steps required to complete a User Story.

---

## 2. Best Practices for Writing User Stories

### A. Title & Structure
- Write titles from the perspective of the user or persona.
- Standard Format: *As a [type of user], I want [an action/goal] so that [a benefit/value].*

### B. Description Field
- Explain **WHO** the feature is for, **WHAT** they want to accomplish, and **WHY**.
- Avoid detailing technical implementation here (leave technical steps for Tasks).
- Include non-goals and constraints to prevent scope expansion.

### C. Acceptance Criteria (AC)
- Acceptance Criteria define the specific conditions that must be satisfied before a User Story can be marked as Closed.
- Must be written **before** development begins.
- Format using testable criteria or Gherkin style:
  - `Given` [initial context]
  - `When` [action is taken]
  - `Then` [expected outcome]
- Include happy paths, edge cases, error conditions, and operational/logging requirements.

### D. Core User Story Fields
- **Value Area:** Specify `Business` (customer value) or `Architectural` (technical foundation).
- **Story Points:** Relative effort estimation for the story.
- **Priority:** 
  - `1`: Product cannot ship without it.
  - `2`: Required, but not immediate.
  - `3`: Optional based on resources/risk.

---

## 3. Best Practices for Defining Tasks

### A. Purpose & Scope
- Tasks break down a User Story into developer-level implementation steps (e.g., database updates, API endpoints, UI components, unit tests).
- Each Task should represent work executable by a single person.

### B. Core Task Fields
- **Title:** Concise, action-oriented implementation detail (e.g., *Implement POST /carrier-reports endpoint*).
- **Activity:** Categorize task type (Development, Testing, Design, Documentation).
- **Original Estimate:** Initial estimated effort in hours.
- **Remaining Work:** Dynamically updated remaining hours as work progresses.

---

## 4. Azure Boards Linking & Hierarchy Rules

1. **Parent-Child Integrity:**
   - Tasks **MUST** be child items linked to a User Story parent.
   - Tasks cannot be parents to other Tasks.
2. **Assignee Rule:**
   - The `Assigned To` field accepts only one user per work item. If multiple developers share work on a story, break it into individual Tasks assigned per developer.
3. **Traceability:**
   - Always link Pull Requests and Commits directly to the Work Item ID in the Development section.
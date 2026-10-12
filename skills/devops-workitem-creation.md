---
name: devops-workitem-creation
description: Enforces step-by-step validation, preview reviews, and safe execution rules for creating User Stories and Tasks via the workflow tool.
---

# Purpose
You manage the transactional creation of Azure DevOps work items for a single preconfigured project.

# Execution Rules
1. **Interactive Inputs:** 
   - Ask for Work Item Type (`User Story` or `Task`), Title, and Description.
   - If Type is `Task`, explicitly ask for the `Parent Story ID`.
2. **Parent Validation:** For Tasks, rely on the backend workflow tool to ensure the parent exists and is a `User Story`. 
3. **Mandatory Review & Approval:** 
   - Never call the creation tool directly without showing a text preview summary first.
   - Force the user to choose: `Approve and create`, `Reject and cancel`, or `Edit the details`.
4. **Result Handling:**
   - On success (`status == "Success"`), display the returned ID and direct URL.
   - On failure (`status == "Failed"`), explain the error message and offer to let the user correct the input.
-- name: azure-devops-workitem-agent
-- description: Handles creation, validation, and comment management for Azure DevOps User Stories and Tasks. Validates parent-child relationships and parses Teams group chat context.


# Azure DevOps Work Item Skill

## Tool Inventory
- `GetWorkItemDetails`: Fetches details for a specific ID (`organization`, `project`, `id`).
- `CreateWorkItem`: Creates a work item (`organization`, `project`, `workItemType`, `title`, `description`, `assignedTo`, `state`, `linkUrl`, `linkType`).
- `AddWorkItemComment`: Posts a discussion comment (`organization`, `project`, `id`, `text`).

Default Configuration:
- `organization`: "nimbusinsights"
- `project`: "AI agents"

---

## Block 1: Parameter Normalization
Before invoking tools, transform all user inputs into standardized formats:

1. **WorkItemType Rules:**
   - `"task"`, `"Task"`, `"TASK"` $\rightarrow$ `"Task"`
   - `"user story"`, `"User Story"`, `"story"`, `"US"` $\rightarrow$ `"User Story"`
2. **Numeric ID Rule:**
   - Convert formatted string IDs like `"#2211"` or `"ID: 2211"` into clean integer numbers: `2211`.
3. **Assignee Mapping:**
   - Convert informal names (e.g., "Assign to Priya") into corporate principal emails (e.g., `Priya@companyname.in`).

---

## Block 2: Workflows

### Module A: Create User Story
1. **Extract Parameters:** `title` (required), `description` (optional), `assignedTo` (optional), `comment` (optional).
2. **Create Item:** Call `CreateWorkItem` with `workItemType = "User Story"`.
3. **Post Comment:** If `comment` is present, call `AddWorkItemComment` using the newly created item `id`.
4. **Output:** Return a summary card containing Item ID, Title, Assignee, and direct link URL.

### Module B: Create Task (With Parent Validation)
1. **Parent Check:** Ensure a `parentStoryId` is supplied. If missing, request it from the user before calling tools.
2. **Parent Validation Step (`GetWorkItemDetails`):**
   - Execute `GetWorkItemDetails` with `id = parentStoryId`.
   - Inspect `body/fields/System_WorkItemType`:
     - **Match (`"User Story"`):** Proceed to Step 3.
     - **Mismatch (Not `"User Story"`):** Stop execution and return:
       > ⚠️ **Validation Error:** Work Item `#<parentStoryId>` is a **<ReturnedType>**, not a User Story. Tasks must be attached to a valid User Story parent.
     - **Not Found (404 Error):** Stop execution and return:
       > ❌ **Validation Error:** Parent Work Item `#<parentStoryId>` was not found in project `AI agents`.
3. **Create Task:**
   - Execute `CreateWorkItem` with `workItemType = "Task"`.
   - Set `linkUrl = "https://dev.azure.com/companyname/projectname/_apis/wit/workItems/<parentStoryId>"`
   - Set `linkType = "Hierarchy-reverse"`
4. **Post Comment:** If `comment` is present, call `AddWorkItemComment` for the new Task ID.
5. **Output:** Present confirmation with Task ID, Parent link confirmation, and URL.

---

## Block 3: Context Parser (Teams & Group Chats)
When tagged/mentioned in a Microsoft Teams group chat or channel:
1. **Synthesize History:** Read preceding chat messages in the thread to build a clear, action-oriented `title` and a detailed `description`.
2. **Mention Detection:** If another team member is tagged in the conversation for task assignment, automatically extract their email for `assignedTo`.

---

## Block 4: Error Handling Matrix

| API Exception / Condition | Primary Cause | Friendly Response |
| :--- | :--- | :--- |
| **`404 Not Found`** | Work Item ID does not exist | *"I couldn't locate work item #<ID> in the 'AI agents' project. Please verify the ID."* |
| **`401 / 403 Forbidden`** | Insufficient permissions | *"Your account lacks write permissions in Azure DevOps for this Area Path."* |
| **Missing Parameter** | `title` is missing | *"Please provide a title for the work item."* |
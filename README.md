# Azure DevOps Agent (Copilot Studio) 🚀

An enterprise-grade, single-project **Copilot Studio AI Agent** that combines real-time Azure Boards knowledge retrieval, live work item lookup, and a multi-step transactional creation workflow for **User Stories** and **Tasks** with parent validation.

---

## 🌟 Key Features

- **Single-Project Focus:** Pre-configured for a dedicated Azure DevOps organization and project. Users never have to enter organization or project names manually.
- **Agile Knowledge Grounding:** Uses custom knowledge sources (`azure-boards-agile-guide.md`) to coach users on writing User Stories, Acceptance Criteria (Gherkin style), and Task breakdowns.
- **Live Q&A Lookups:** Dynamically queries Azure Boards using optimized read tools (`Get work item details V2`) to answer real-time questions about existing work items.
- **Transactional Creation Workflow:** Enforces a strict 4-step execution model (**Collect Inputs** $\rightarrow$ **Parent Validation** $\rightarrow$ **User Preview & Explicit Approval** $\rightarrow$ **Execution**).
- **Automated Hierarchy Validation:** Ensures Tasks are created only under verified `User Story` parents to preserve backlog integrity.

---

## 🏗️ Architecture Overview

```text
                               ┌──────────────────────────────────────────────┐
                               │       Copilot Studio Orchestrator            │
                               │             (Claude Sonnet / GPT-4o)         │
                               └──────────────────────┬───────────────────────┘
                                                      │
                       ┌──────────────────────────────┼──────────────────────────────┐
                       ▼                              ▼                              ▼
      ┌────────────────────────────────┐  ┌───────────────────────┐  ┌───────────────────────────────┐
      │      Knowledge Grounding       │  │    Real-Time Read     │  │    Transactional Workflow     │
      │   azure-boards-agile-guide.md  │  │ Get work item details │  │ Create-AzureDevOps-WorkItem   │
      └────────────────────────────────┘  └───────────────────────┘  └───────────────────────────────┘
                                                                                     │
                                                                   ┌─────────────────┴─────────────────┐
                                                                   ▼                                   ▼
                                                      [User Story Creation Path]           [Task Creation Path]
                                                                   │                                   │
                                                                   ▼                                   ▼
                                                          Create Work Item              Validates Parent via Read
                                                           (User Story)                                │
                                                                                                       ▼
                                                                                              Check Parent == "User Story"
                                                                                                       │
                                                                                       ┌───────────────┴───────────────┐
                                                                                       ▼                               ▼
                                                                             [Valid: Create Task]    [Invalid: Return Error]
```

---

## 🛠️ Prerequisites

1. **Microsoft Copilot Studio** license and environment with Generative AI features enabled.
2. **Power Automate / Cloud Flow** maker permissions.
3. **Azure DevOps** account with permissions to read/create work items in your target project.
4. An established Azure DevOps connection using OAuth / PAT.

---

## 🚀 Step-by-Step Deployment Guide

### Step 1: Prepare Knowledge Base (`azure-boards-agile-guide.md`)

Create a local markdown file named `azure-boards-agile-guide.md` with your organization's Agile rules and official Azure Boards best practices:

```markdown
# Azure Boards Agile Work Item & Best Practices Guide

## 1. Hierarchy & Work Item Types (WITs)
- **User Story:** Represents specific end-user requirements scoped to a single sprint/iteration.
- **Task:** Represents granular technical implementation steps required to complete a User Story.

## 2. Best Practices for Writing User Stories
- Format: *As a [type of user], I want [an action/goal] so that [a benefit/value].*
- Acceptance Criteria format:
  - `Given` [initial context]
  - `When` [action is taken]
  - `Then` [expected outcome]

## 3. Azure Boards Linking & Hierarchy Rules
- Tasks **MUST** be child items linked to a valid User Story parent.
```

---

### Step 2: Build the Power Automate Creation Workflow

Create an automated Cloud Flow named **`Create-AzureDevOps-WorkItem`**.

#### 1. Flow Trigger: 

 ## A. *Trigger Type:* : When an agent calls the workflow
Define the following input variables:
- `workItemType` Type of work item to create: User Story or Task
- `title` The title of the work item
- `description` Detailed description and acceptance criteria
- `parentStoryId` Optional parent story ID (required if item type is Task)

## B. *If/Else*

 # [ Condition 1 ] ── Is workItemType == "Task"?

- `property` workItemType (insert dynamic content)
- `operator` Equals (AND)
- `value` Task
- `run after` check Failed, TimedOut, Skipped

*If True* : Get work item details (add tool)

configure the following
- `organization name`: <your-org-name>
- `project name`: <azure-boards-prj-name>
- `work item type`: User Story (please check this)
- `work item ID`: parentStoryID (insert dynamic content)

*Else* : Create a work item (add tool)

configure the following
- `organization name`: <your-org-name>
- `project name`: <azure-boards-prj-name>
- `work item type`: User Story
- Work item details:
  - `Title` title (insert dynamic content)
  - `Description` description (insert dynamic content)

*Respond to the agent*

 # [ Condition 2 ] ── to Get work item details (If True) add If/Else

- `property` workItemType (insert dynamic content)
- `operator` Equals (AND)
- `value` User Story

*If True* : create a work item - Task (add tool)

configure the following
- `organization name`: <your-org-name>
- `project name`: <azure-boards-prj-name>
- `work item type`: Task
- Work item details:
  - `Title` title (insert dynamic content)
  - `Description` description (insert dynamic content)
  *Respond to the agent*

*Else*:  *Respond to the agent*



#### 2. Decision Logic Canvas Setup

```text
[ Trigger: When an agent calls the workflow ]
               │
               ▼
       [ Condition 1 ] ── Is workItemType == "Task"?
               │
               ├─► NO (User Story)
               │      │
               │      ▼
               │   [ Action: Create a work item ] (Type: User Story)
               │      │
               │      ▼
               │   [ Respond to the agent #1 ]
               │      • status = "Success"
               │      • createdUrl = Output.Url
               │      • message = "User Story created successfully."
               │
               └─► YES (Task)
                      │
                      ▼
            [ Action: Get work item details (V2) ] (ID: parentStoryId)
                      │
                      ▼
               [ Condition 2 ] ── Is Work Item Type == "User Story"?
                      │
                      ├─► NO (Invalid Parent)
                      │      │
                      │      ▼
                      │   [ Respond to the agent #2 ]
                      │      • status = "Failed"
                      │      • createdUrl = ""
                      │      • message = "Validation Error: Parent ID is not a valid User Story."
                      │
                      └─► YES (Valid Parent)
                             │
                             ▼
                          [ Action: Create a work item ]
                             • Work Item Type: Task
                             • Link URL: https://dev.azure.com/{Org}/{Project}/_apis/wit/workitems/{parentStoryId}
                             • Link Type: Parent
                             │
                             ▼
                          [ Respond to the agent #3 ]
                             • status = "Success"
                             • createdUrl = Output.Url
                             • message = "Task created successfully under Parent Story."
```

---

### Step 3: Configure Agent in Copilot Studio

1. Open **Copilot Studio** and select **+ Create Agent**.
2. **Agent Name:** `Azure DevOps Agent`
3. **Model Selection:** Select **Claude Sonnet** or **GPT-4o** under **Settings $\rightarrow$ Generative AI** (recommended for strict tool orchestration and instruction adherence).

#### System Instructions

Paste the following master prompt into the **Instructions** box:

```markdown
---
name: azure-devops-agent
description: Expert Azure DevOps assistant for single-project Agile Q&A, knowledge grounding, and validated work item creation.
---

# Role & Core Purpose
You are an expert Agile Process Coach and Azure DevOps Assistant for a single preconfigured project. Your responsibilities are divided into two distinct operational scopes:
1. **Advisory & Q&A:** Answering questions about Agile, User Stories, Tasks, and project standards using attached knowledge and read tools.
2. **Transactional Execution:** Interactively collecting inputs, validating parents, showing a review preview, and calling the workflow tool to create work items.

---

# Scope 1: Agile Q&A and Knowledge Grounding
- **Source Material:** Use the attached `azure-boards-agile-guide.md` knowledge source to answer questions about user story writing, acceptance criteria (Gherkin style), and task definitions.
- **Dynamic Lookups:** If a user asks about specific existing items (e.g., "What is the status of Task #1042?"), invoke the `Get work item details (V2)` tool to fetch real-time data.
- **Boundary:** Do not trigger creation workflows for purely informational or query-based questions.

---

# Scope 2: Work Item Creation Workflow
When a user expresses intent to create a `User Story` or `Task`, strictly execute these 4 steps:

## Step 1: Collect Required Inputs
Interactively gather missing details from the user:
- **Work Item Type:** Must be `User Story` or `Task`.
- **Title:** Summary of the work item.
- **Description:** Details and/or Acceptance Criteria.
- **Parent Story ID:** Required **ONLY** if the item type is `Task`.

## Step 2: Present Preview & Demand Approval
Before calling any creation tool, present a formatted review summary and wait for explicit user interaction:

---
### 📋 Work Item Review
- **Type:** {workItemType}
- **Title:** {title}
- **Description:** {description}
- **Parent User Story:** #{parentStoryId} *(Only if Task)*

Do you approve creating this work item?
- Approve and create
- Reject and cancel
- Edit the details
---

## Step 3: Tool Execution Rules
- If **Reject and cancel**: Respond *"Work item creation canceled."* and stop.
- If **Edit the details**: Prompt which field to change and update it.
- If **Approve and create**: Invoke the `Create-AzureDevOps-WorkItem` workflow tool passing `workItemType`, `title`, `description`, and `parentStoryId`.

## Step 4: Handle Tool Response Outputs
- If workflow output `status == "Success"`: Inform the user with a green confirmation message displaying the returned work item ID and direct web URL.
- If workflow output `status == "Failed"`: Read the workflow's `message` output (such as parent validation failure) and offer to let the user input a valid Parent Story ID or cancel.

---

# Global Constraints
- **Fixed Project Scope:** Never prompt the user for Organization or Project names; these are hardcoded in the backend tools.
- **Safety First:** Rely on the workflow's built-in parent validation before committing Tasks to Azure Boards.
```

---

### Step 4: Connect Knowledge & Tools

1. **Upload Knowledge File:**
   - Go to **Knowledge** $\rightarrow$ **+ Add Knowledge** $\rightarrow$ **Files**.
   - Upload `azure-boards-agile-guide.md`.
   - **Description for AI:** *"Official Azure Boards Agile guidance for writing User Stories, Acceptance Criteria, and Tasks."*

2. **Add Work Item Lookup Tool:**
   - Go to **Tools** $\rightarrow$ **+ Add Tool** $\rightarrow$ Select `Get work item details (V2)`.
   - Set **Organization** and **Project** inputs to your hardcoded target project values.
   - Set Tool Description:
     ```text
     Fetches details (title, description, acceptance criteria, type) of an Azure DevOps work item by ID. Used for answering Q&A about existing items or validating parent User Stories for Tasks.
     ```

3. **Add Work Item Creation Workflow Tool:**
   - Go to **Tools** $\rightarrow$ **+ Add Tool** $\rightarrow$ Select your Power Automate Flow `Create-AzureDevOps-WorkItem`.
   - Set Tool Description:
     ```text
     Creates a User Story or Task in Azure DevOps after performing parent validation. Requires explicit user approval before execution.
     ```

---

## 🧪 Testing & Verification

| Test Scenario | User Input Example | Expected Behavior |
| :--- | :--- | :--- |
| **Agile Q&A** | *"How should I write acceptance criteria for a login story?"* | Answers using Gherkin style rules from `azure-boards-agile-guide.md`. No workflow called. |
| **Live Item Q&A** | *"What is the status of story #1042?"* | Invokes `Get work item details (V2)` and summarizes state and title. |
| **Task Creation (Valid Parent)** | *"Create a task to build database index for story #4250."* | Asks for missing inputs, renders preview card, waits for explicit approval, and creates Task. |
| **Task Creation (Invalid Parent)** | *"Create a task under parent #9999."* | Executes workflow, fails parent validation step, and returns clean error message asking for a valid User Story ID. |

---

## 📚 References & Supporting Documentation

- [Microsoft Copilot Studio Documentation](https://learn.microsoft.com/en-us/microsoft-copilot-studio/)
- [Configure Agent Details & Instructions in Copilot Studio](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agents-experience/authoring-instructions)
- [Azure DevOps Services REST API Documentation](https://learn.microsoft.com/en-us/rest/api/azure/devops/)
- [Azure DevOps Connector in Power Automate](https://learn.microsoft.com/en-us/connectors/azuredevops/)
- [Azure Boards Agile Process Work Item Types](https://learn.microsoft.com/en-us/azure/devops/boards/work-items/about-work-items)

---

## 📄 License

This repository is licensed under the [MIT License](LICENSE).
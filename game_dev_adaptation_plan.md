# Plan: Adapting Roo Code Memory Bank for Game Development Workflow

**Version:** 1.0 (Approved 2025-04-12)

**Goal:** Modify the existing Roo Code Memory Bank configuration (`.clinerules-*` files) and Memory Bank structure to align with the specified game development workflow, incorporating validation steps, issue tracking, failure loop detection, and architectural guardrails.

---

## 1. Memory Bank Structure Modifications

**Action:** Add two new core files to the `memory-bank/` directory and define their initial content structure within the relevant `.clinerules-*` files.

### 1.1. Add `memory-bank/issuesLog.md`

- **Purpose:** Systematically log reported issues (bugs, problems), investigation steps, root causes, solutions implemented, and validation status.
- **Implementation:** Modify the `memory_bank_strategy.initialization.initial_content` section in all `.clinerules-*` files to include the creation logic and initial content for this file.
- **Initial Content Structure (to be added to `.roorules-*` initial_content):**

  ```markdown
  # Issues Log

  This file tracks reported issues, their analysis, and resolution status.
  YYYY-MM-DD HH:MM:SS - Log initialized.

  ---

  ## Issue ID: [Unique ID, e.g., BUG-001]

  - **Reported:** YYYY-MM-DD HH:MM:SS by [User/Mode]
  - **Status:** [Open | Investigating | Resolved | Closed]
  - **Severity:** [Critical | High | Medium | Low]
  - **Description:** [Detailed description of the issue]
  - **Reproduction Steps:** [Steps to reproduce the issue]
  - **Investigation Notes:**
    - [YYYY-MM-DD HH:MM:SS] - [Note from Debug/Code/etc. mode]
  - **Resolution:** [Description of the fix implemented]
  - **Validation:** [Passed | Failed] - YYYY-MM-DD HH:MM:SS by [User/Mode]

  ---

  _(Repeat structure for each new issue)_
  ```

### 1.2. Add `memory-bank/gameDesignDoc.md`

- **Purpose:** Store specific game design details (mechanics, narrative, levels) separate from the high-level `productContext.md`.
- **Implementation:** Modify the `memory_bank_strategy.initialization.initial_content` section in all `.clinerules-*` files to include the creation logic and initial content for this file.
- **Initial Content Structure (to be added to `.roorules-*` initial_content):**

  ```markdown
  # Game Design Document

  This file contains detailed game design specifications.
  YYYY-MM-DD HH:MM:SS - Log initialized.

  ## Core Gameplay Loop

  - [Description]

  ## Key Mechanics

  - **[Mechanic 1]:** [Description]
  - **[Mechanic 2]:** [Description]

  ## Level Design Concepts

  - **[Level 1]:** [Description]

  ## Narrative Outline

  - [Synopsis]

  _(Structure to be expanded as needed)_
  ```

### 1.3. Tailor Existing File Prompts

- **Action:** Modify the existing `initial_content` definitions for `productContext.md`, `activeContext.md`, and `progress.md` within all `.clinerules-*` files.
- **Goal:** Make the initial placeholder text more relevant to game development (e.g., prompt for genre, platform, core mechanics in `productContext.md`).

### 1.4. Add Environment Details to `productContext.md`

- **Action:** Add a dedicated section to the `initial_content` definition for `productContext.md` within all `.clinerules-*` files.
- **Goal:** Capture essential technical context early in the project.
- **New Section Structure (to be added within `productContext.md` initial content):**

```markdown
## Development Environment & Engine

- **Game Engine:** [Engine Name, e.g., Godot, Unity, Unreal, Custom]
- **Engine Version:** [Specific Version, e.g., 4.2.1, 2022.3.5f1]
- **Primary Language(s):** [e.g., GDScript, C#, C++, JavaScript]
- **Target Platform(s):** [e.g., PC (Windows/Mac/Linux), Web, Mobile (iOS/Android)]
- **Version Control System:** [e.g., Git, SVN, Perforce, None]
- **Key Tools/SDKs:** [e.g., Blender, FMOD, Steam SDK]
```

---

## 2. Mode Adaptation & Workflow Modifications (via `.clinerules-*` files)

**Action:** Modify the identity, collaboration rules, memory bank interactions, and add specific workflow logic to each relevant `.clinerules-*` file.

### 2.1. General Changes (Apply to all relevant modes)

- **Memory Bank Reading:** Ensure all modes read the new `issuesLog.md` and `gameDesignDoc.md` during initialization (`if_memory_bank_exists` step).
- **Memory Bank Updates:** Define triggers and actions (`memory_bank_updates` section) for interacting with the new files (e.g., Debug mode creating entries in `issuesLog.md`).

### 2.2. Architect Mode (`.roorules-architect`) - "Game PM" Role

- **Identity:** Update `description` to include "Game Project Management" responsibilities (task oversight, validation coordination).
- **Workflow - Initialization Environment Check:**
  - Add logic: After Memory Bank initialization/loading (`if_memory_bank_exists`), _immediately_ check `productContext.md` for the "Development Environment & Engine" section.
  - If section is missing or incomplete: Use `ask_followup_question` to prompt the user for Game Engine (Name & Version), Primary Language(s), Target Platform(s), and Version Control System.
  - Update `productContext.md` with the provided information _before_ proceeding with any other task.
- **Workflow - Validation Step:**
  - Add logic: When another mode (esp. Code) signals task completion requiring validation, Architect mode must be activated.
  - Architect mode reviews context (user feedback, Test mode results).
  - Architect mode confirms validation status (Pass/Fail).
  - If Fail: Increment failure counter (see section 3), provide feedback, hand back/escalate.
  - If Pass: Update `progress.md` and potentially `issuesLog.md` validation status.
- **Workflow - Architectural Guardrail (Review):**
  - Add logic: When receiving a handoff triggered by a "Potentially Significant Architectural Change" (see section 4):
    - Analyze the proposed change against Memory Bank docs (`productContext.md`, `decisionLog.md`, etc.).
    - If minor/planned: Approve, document, hand back to proceed.
    - If significant/unplanned: Analyze implications, consult user via `ask_followup_question`, act on approval/rejection (document decision, instruct original mode).
- **Workflow - Failure Loop Escalation:**
  - Add logic: When receiving a handoff due to "Recurring failure threshold reached":
    - Review context (`activeContext.md`, history, logs).
    - Decide next step: Web search via Ask, deeper debugging via Debug, re-evaluate strategy with user.
    - Instruct the chosen mode.
    - Reset failure counter in `activeContext.md` _only_ when providing fundamentally new guidance/information.
- **Memory Bank:** Add rules for reading/reviewing `issuesLog.md` and `gameDesignDoc.md`.

### 2.3. Code Mode (`.roorules-code`) - "Game Programmer" Role

- **Identity:** Update `description` to "Game Programmer".
- **Workflow - User Testing Prompt:**
  - Add logic: _Before_ using `attempt_completion` for a feature/fix, Code mode _must_ use `ask_followup_question` to request user testing and confirmation.
- **Workflow - Failure Handling:**
  - Add logic: On receiving user feedback indicating failure OR notification of failed PM validation:
    - Increment failure counter in `activeContext.md` (see section 3).
    - Check threshold. If reached, escalate to Architect/Game PM. If not, attempt fix.
- **Workflow - Architectural Guardrail (Check):**
  - Add logic: _Before_ calling file modification tools (`apply_diff`, `write_to_file`), check if the change matches defined architectural triggers (see section 4).
  - If yes: Halt modification, hand off to Architect/Game PM for review.
  - If no: Proceed with modification.
- **Memory Bank:** Add rules to read `gameDesignDoc.md` for guidance and `issuesLog.md` when fixing bugs.

### 2.4. Ask Mode (`.roorules-ask`) - "Game Documentation & Knowledge" Role

- **Identity:** Update `description` to "Game Documentation & Knowledge".
- **Workflow - Web Search:**
  - Add/refine logic: Accept handoffs from Architect/Game PM for web searches related to documentation, general knowledge, tutorials, API references using the `mcp_openai_server` (`get_openai_response` tool).
- **Memory Bank:** Add rules to read `gameDesignDoc.md` and `issuesLog.md` to answer relevant questions.

### 2.5. Debug Mode (`.roorules-debug`) - "Game Debugger" Role

- **Identity:** Update `description` to "Game Debugger".
- **Workflow - Failure Handling:**
  - Add logic: When working on an issue from `issuesLog.md`, if a fix attempt is validated and found unsuccessful:
    - Increment failure counter in `activeContext.md` (see section 3).
    - Check threshold. If reached, escalate to Architect/Game PM. If not, continue debugging.
- **Workflow - Web Search:**
  - Add/refine logic: Accept handoffs from Architect/Game PM for web searches related to specific technical errors, library issues, or problem-solving using the `mcp_openai_server` (`get_openai_response` tool).
- **Workflow - Architectural Guardrail (Check):**
  - Similar to Code mode, add a check before file modifications for potential architectural changes, triggering review if needed.
- **Memory Bank:** Add rules/triggers for creating/updating entries in `issuesLog.md` (reporting, investigation notes, resolution details).

### 2.6. Test Mode (`.roorules-test`)

- **Workflow - Validation Input:**
  - Refine logic: Ensure Test mode results (pass/fail, coverage) are clearly reported and can be used by Architect/Game PM during the validation step.
- **Memory Bank:** Add rules to potentially update the validation status in `issuesLog.md` upon successful test runs confirming a fix.

---

## 3. Failure Loop Detection & Escalation

- **State Tracking:** Implement tracking of `Consecutive Failed Attempts (Validation/Fix)` within `activeContext.md` for the current primary task/issue.
- **Increment Triggers:** Define rules for Code, Debug, and Architect modes to increment this counter upon failed validation/fix attempts (from user, PM, or testing).
- **Threshold:** Set the failure threshold to **3**.
- **Escalation:** Implement logic in Code and Debug modes to check the threshold after incrementing the counter. If reached, trigger an immediate handoff to Architect/Game PM with specific context ("Recurring failure threshold reached...").
- **PM Action:** Architect/Game PM analyzes the escalation, decides the next course of action (web search via Ask, deeper debugging, strategy review), and resets the counter _only_ when providing new guidance.

---

## 4. Architectural Change Guardrail

- **Define Triggers:** Identify patterns within `.roorules-*` that constitute "Potentially Significant Architectural Changes":
  - Modifying dependency files (e.g., `package.json`, engine project files).
  - Modifying files identified as core architectural components.
  - Major file structure changes (top-level directories).
  - Changes to public APIs/interfaces of core modules.
- **Pre-Change Check:** Implement logic in Code and Debug modes to check for these triggers _before_ executing file modifications.
- **Review Handoff:** If a trigger is matched, halt the modification and hand off to Architect/Game PM for review, providing the proposed change and context.
- **PM Review:** Architect/Game PM analyzes the change against Memory Bank docs, determines significance/alignment, consults the user via `ask_followup_question` if significant and unplanned, and instructs the original mode based on approval/rejection.

---

**End of Plan**

# PayLink Project — AI Prompt Log

This document tracks all AI interactions across the PayLink project lifecycle, as required by the project specification and course guidelines.

## Logging Guidelines for Team Members

- **When to Log:** Every team member must record AI interactions when an AI tool generates more than half of a change, or when an AI suggestion is analyzed, rejected, or modified.
- **Commit Requirement:** When committing AI-assisted work, cite the tool name in the commit message (e.g., `AI-assisted: Claude 3.5 Sonnet` or `AI-assisted: ChatGPT-4o`).
- **Transparency:** Honest logging of rejected or corrected answers is required and earns marks when properly documented and analyzed.

---

## Prompt Log Table

| Date       | Author         | Stage                    | Tool & Model                    | Prompt                                                                                                                                                                                 | Summary of Answer                                                                                                              | Verdict & Why                                                                                                                            |
| ---------- | -------------- | ------------------------ | ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-11-23 | Ebrahem Hassan | Design README creation   | GitHub Copilot (Copilot Fusion) | Create `docs/design/README.md` documenting the confirmed technology stack, Modular Monolith with Clean Architecture, eight modules, architectural rules, and planned design documents. | Generated the initial PayLink Design Overview, covering the technology stack, architecture, modules, and design documentation. | **Fixed** – The initial version needed consistent module names, explicit data ownership boundaries, and Markdown formatting corrections. |
| 2026-11-23 | Ebrahem Hassan | Design README refinement | GitHub Copilot (Copilot Fusion) | Refine the README by standardizing module names, adding initial data ownership for all eight modules, correcting Markdown formatting, and preserving existing decisions.               | Updated the README with consistent module names and an initial module data ownership table.                                    | **Accepted** – The revised content was reviewed and aligned with the agreed architectural decisions and Phase 1 scope.                   |
| 2026-11-23 | Ebrahem Hassan | Use Case Diagram – Actor Identification | ChatGPT (GPT-6) | Identify all actors involved in the PayLink system based on the functional requirements. | Proposed the initial actors and their responsibilities, distinguishing external actors from internal system modules. | **Fixed** – Reviewed actor responsibilities and refined the grouping to avoid unnecessary actors. |
| 2026-11-24 | Ebrahem Hassan | Domain Class Diagram Creation | GitHub Copilot (Copilot Fusion) | Create the PayLink Domain Class Diagram using PlantUML, organizing domain classes into modules, defining attributes and relationships with multiplicities, and documenting important business rules. | Created the PayLink domain class diagram in PlantUML with module packages, domain classes, attributes, relationships, multiplicities, and business-rule notes. | **Fixed** – The diagram was reviewed and corrected to align multiplicities and agent-assisted commission behavior with the domain decisions. |

---

## Instructions for Appending Entries

Add new rows to the bottom of the table using the following column order:

1. **Date** (`YYYY-MM-DD`)
2. **Author** (Member A, B, C, or D)
3. **Stage** (e.g., `Phase 1 - Requirements`, `Phase 1 - DevOps`, `Phase 2 - Accounts`, etc.)
4. **Tool & Model** (e.g., `ChatGPT-4o`, `Claude 3.5 Sonnet`, `GitHub Copilot`)
5. **Prompt** (The exact prompt or structured prompt template used)
6. **Summary of Answer** (Brief summary of what the AI returned)
7. **Verdict & Why** (`ACCEPTED`, `FIXED`, or `REJECTED`, with a concrete technical explanation)

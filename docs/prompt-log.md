# PayLink Project — AI Prompt Log

This document tracks all AI interactions across the PayLink project lifecycle, as required by the project specification and course guidelines.

## Logging Guidelines for Team Members

- **When to Log:** Every team member must record AI interactions when an AI tool generates more than half of a change, or when an AI suggestion is analyzed, rejected, or modified.
- **Commit Requirement:** When committing AI-assisted work, cite the tool name in the commit message (e.g., `AI-assisted: Claude 3.5 Sonnet` or `AI-assisted: ChatGPT-4o`).
- **Transparency:** Honest logging of rejected or corrected answers is required and earns marks when properly documented and analyzed.

---

## Prompt Log Table

| Date       | Author   | Stage            | Tool & Model      | Prompt                                                                                                                                                                                                                                                               | Summary of Answer                                                                                                                                               | Verdict & Why                                                                                                                                                                                                                                                                                                                                                                     |
| ---------- | -------- | ---------------- | ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-11-23 | Member D | Phase 1 (Design) | Claude 3.5 Sonnet | "Generate PlantUML state diagram code (`state-payment.puml`) for the PayLink payment lifecycle. State transitions: Initiated, FundsHeld, SentToBiller, Completed, Failed, Unknown, Rejected, Refunded. Include transitions for timeouts and failed payment refunds." | Provided a PlantUML state diagram containing 8 states, but included a direct transition from `Unknown` to `Refunded`, and marked `Unknown` as a terminal state. | **REJECTED & FIXED**<br>The AI violated Section 2.5 of the project brief: `Unknown` cannot transition directly to `Refunded` (it must resolve to `Completed` or `Failed` first, and only `Failed` can transition to `Refunded`). The prompt was refined to explicitly enforce strict state transition rules and list exact terminal states (`Completed`, `Rejected`, `Refunded`). |

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

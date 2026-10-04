# AI Is Wrong

Use this family when an AI-produced answer, classification, or inference is incorrect for the available evidence or task.

## What engineers notice

- answers conflict with source material or known facts
- the system invents unsupported details
- accuracy changes sharply across similar inputs

## Questions to ask first

1. Was the required context available, relevant, and current?
2. Is the failure in retrieval, interpretation, instruction, or evaluation?
3. Which parts of the outcome require deterministic rules rather than inference?

## Cases

- [Eligibility translation drifted at runtime](../something-is-nondeterministic/cases/llm-eligibility-drift.md) - known inputs re-derived by a model drift; freeze verified answers instead.

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

- [The text checker fixed the typos it was supposed to catch](cases/text-checker-hides-typos.md) - a checker that autocorrects cannot verify exact text; fail only when independent checkers agree, and treat disagreement as "unverified", never as a pass.

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

## Related patterns

- [Independent Checker Agreement](../../patterns/independent-checker-agreement.md)

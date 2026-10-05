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

- [The prompt's formatting ended up in the image](cases/prompt-formatting-in-image.md) - an image model draws marker characters placed around the text it must render; keep formatting out of the text itself, and make the checker count extra characters instead of cleaning them away.

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

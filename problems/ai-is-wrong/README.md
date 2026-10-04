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

- [A text reader that corrects typos cannot verify exact text](cases/text-reader-that-corrects-typos-cannot-verify-text.md) - a reader biased toward real words cannot pass exact text; require two independent readers to agree before failing, and mark disagreement as unverified.

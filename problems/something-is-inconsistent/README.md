# Something Is Inconsistent

Use this family when related records, views, systems, or decisions disagree when they should represent the same reality.

## What engineers notice

- users receive conflicting answers from different parts of a system
- duplicate processing creates competing records
- a sequence of updates leaves an invalid intermediate state

## Questions to ask first

1. What invariant or relationship should always hold?
2. Which write, read, or asynchronous update first violated it?
3. Is the disagreement temporary, expected, or permanently incorrect?

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

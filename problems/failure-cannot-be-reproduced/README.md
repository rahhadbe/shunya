# Failure Cannot Be Reproduced

Use this family when an observed failure cannot be recreated well enough to understand, verify, or fix it.

## What engineers notice

- logs describe the error but not the conditions that caused it
- a retry succeeds without explaining the original failure
- production behavior differs from local or test environments

## Questions to ask first

1. Which input, state, timing, version, and dependency details are missing?
2. What evidence survives the failure, and what is lost?
3. Can the operation be replayed or simulated without repeating harmful side effects?

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

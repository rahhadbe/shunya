# Data Is Stale

Use this family when a decision or user experience relies on information that is older than the situation allows.

## What engineers notice

- users see values that disagree with a recent change
- different systems report different versions of the same fact
- the issue resolves after waiting or refreshing

## Questions to ask first

1. What is the source of truth, and when was it last updated?
2. Where does delay enter: caching, replication, queuing, or a failed update?
3. How fresh must this data be for the affected decision?

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

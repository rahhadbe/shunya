# Change Is Dangerous

Use this family when a deployment, migration, configuration change, or dependency update has an unacceptable chance of causing harm.

## What engineers notice

- a small change affects an unexpectedly broad area
- rollback is slow, incomplete, or impossible
- the change behaves differently across environments

## Questions to ask first

1. What could fail, and which users, systems, or data would be affected?
2. Can the change be observed, limited, and reversed safely?
3. Which assumptions need an experiment or staged rollout before full adoption?

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

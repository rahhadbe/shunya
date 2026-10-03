# Something Is Unreliable

Use this family when a system intermittently fails, degrades, or needs repeated manual recovery during normal operation.

## What engineers notice

- retries sometimes succeed without an explained cause
- error rates rise around dependency failures, load, or particular inputs
- an operation completes only after human intervention

## Questions to ask first

1. What is the failure boundary, and which component first reports it?
2. Is the failure transient, permanent, or caused by an invalid request or state?
3. Can retry be made safe and observable, or does recovery need a different action?

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

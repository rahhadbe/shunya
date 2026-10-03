# Something Is Failing Halfway

Use this family when a multi-step operation stops after some effects have already occurred.

## What engineers notice

- one system reports success while another did not complete
- a retry creates duplicate work or charges
- manual recovery is needed after a timeout or dependency failure

## Questions to ask first

1. Which steps completed, and which effects are durable?
2. What should a retry, rollback, or compensating action do from the current state?
3. Can each step be identified and repeated safely?

## Related patterns

- [Idempotent Retry](../../patterns/concurrency/idempotent-retry.md)

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

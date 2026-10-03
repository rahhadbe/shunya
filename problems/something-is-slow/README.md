# Something Is Slow

Use this family when a request, workflow, build, query, or system takes longer than its users or constraints allow.

## What engineers notice

- latency rises while throughput falls or stays flat
- average latency looks acceptable but tail latency is high
- the problem appears only under load or for a subset of requests

## Questions to ask first

1. What changed, and what boundary should be measured?
2. Is the bottleneck CPU, I/O, network, contention, queuing, or an external dependency?
3. Does the average hide a tail-latency or concurrency problem?

## Related patterns

- [Reduce Lock Duration](../../patterns/concurrency/reduce-lock-duration.md)

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

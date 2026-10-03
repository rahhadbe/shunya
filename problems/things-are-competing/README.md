# Things Are Competing

Use this family when multiple operations need the same limited resource and progress slows, fails, or becomes unfair.

## What engineers notice

- latency or queue depth rises only under concurrent load
- lock waits, rate limits, or pool exhaustion increase
- a small number of resources receive disproportionate traffic
- requests succeed when retried but not on the first attempt

## Questions to ask first

1. What resource is actually contested: a lock, connection, queue, CPU, rate limit, or partition?
2. Who acquires it, in what order, and for how long?
3. Is contention broad, or concentrated around a hot resource?

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson. A pattern can be promoted later when multiple cases support it.

## Related families

- [Something Is Slow](../something-is-slow/README.md)
- [Something Is Unreliable](../something-is-unreliable/README.md)
- [Something Is Nondeterministic](../something-is-nondeterministic/README.md)

# Something Is Nondeterministic

Use this family when the same input, state, or workflow can produce different outcomes.

## What engineers notice

- results change across identical or near-identical runs
- failures appear only with particular timing or ordering
- probabilistic output owns a decision that must be repeatable

## Questions to ask first

1. Is the variation inherent to the task or an accidental property of the system?
2. Which input, state, timing, version, or dependency differs between outcomes?
3. Where does a deterministic boundary or ordering constraint belong?

## Cases

- [Eligibility translation drifted at runtime](cases/llm-eligibility-drift.md) - serve verified answers from a lookup; call the model only for unseen inputs.

## Related patterns

- [The Crystallization Pattern](../../patterns/crystallization-pattern.md)

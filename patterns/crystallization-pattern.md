# The Crystallization Pattern

## Intent

Make a probabilistic decision repeatable by serving verified answers from an authoritative
lookup, and invoking the probabilistic solver (typically an LLM) only for inputs not yet
verified: discover once, verify, freeze, reuse.

## The idea

Many tasks map **unbounded free-form input** to a **finite, structured output** — a
category, a route, a schema object, a logic tree. Calling a model on every request makes the
whole path probabilistic: the same input can yield a different output, and latency and cost
are paid to re-derive answers that already exist.

Crystallization inverts the control flow:

```text
Input arrives → normalise → look it up in a verified map (input → output)
  ├─ HIT  → return the frozen answer (deterministic, fast, verified)
  └─ MISS → call the solver → verify the result → write it back → future requests HIT
```

- **A verified answer is a fact you own, not an oracle to re-consult.** Facts belong in a
  lookup, not behind a per-call model with built-in variance.
- **This is not caching.** A cache holds a disposable copy and expires it. Here the stored
  entry is the authoritative source of truth, and the model is the fallback that fills gaps.
- **Keep an explicit "I don't know" state** in the output schema so the solver can decline
  instead of guessing. Unknowns go to review rather than becoming silent errors.
- The name describes the change from fluid (probabilistic output) to solid (deterministic
  lookup). It is a fast, low-variance decision layer in front of a slow, generative one.

## Use it when

- The output space is finite or schema-constrained.
- The same or equivalent inputs recur over time.
- Determinism and correctness matter more than open-ended variety.
- An answer can be verified once and trusted until its inputs or rules change.
- You can afford a verification and write-back step.

## Do not use it when

- Every input is unique and open-ended (free-form writing, open conversation).
- Outputs are subjective or expected to vary per request.
- The correct answer changes faster than entries can be re-verified.
- No trustworthy verification step exists — an unverified map freezes errors permanently.

## Trade-offs

- **Improves:** determinism on verified inputs; correctness on the verified set is asserted
  at write time rather than re-derived at runtime; latency drops to a lookup; solver cost
  falls as recurring traffic is absorbed into the map.
- **Costs:** verification and write-back discipline; the map is only as correct as its
  verification; keys need normalisation, and paraphrases that normalisation misses still
  reach the solver; fuzzy matching to widen hits reintroduces probabilistic behaviour;
  entries need invalidation when schemas or rules change; a new source-of-truth artifact to
  own and govern. It does not improve the solver — it makes it optional for verified inputs.

## Evidence

- [Eligibility translation drifted at runtime — froze verified LLM outputs into a lookup](../problems/something-is-nondeterministic/cases/llm-eligibility-drift.md)
  — an LLM translator plateaued near 97% on a known set; a verified statement → expression
  map with LLM fallback made verified inputs deterministic, with latency and model cost
  falling as traffic recurred.

Currently supported by one case. Independent cases from other contexts (for example ticket
classification, request routing, or tool selection) would strengthen it.

## Related

- [Something Is Nondeterministic](../problems/something-is-nondeterministic/README.md)
- [AI Is Wrong](../problems/ai-is-wrong/README.md)

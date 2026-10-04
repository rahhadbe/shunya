# Eligibility translation drifted at runtime — froze verified LLM outputs into a lookup

## Problem

A runtime component translated free-form, human-written eligibility statements into a
structured, executable logic tree (atoms, operators, values) that a deterministic engine
could act on. A tuned LLM agent performed the translation on every request.

For statements nearly identical to ones already seen, the agent occasionally produced a
*different* result than before: a dropped condition, or an `UNKNOWN` atom for a phrase it
had parsed correctly many times prior. Because the downstream decision was a real
eligibility determination, non-deterministic output was a correctness defect, not a
cosmetic one — the same input could yield a different outcome on a different day.

This mattered most once volume grew and the same statements began recurring at scale.

## Context

- The input space is effectively unbounded (people phrase the same rule many ways); the
  output space is finite and schema-constrained.
- A golden set of ~450 unique statements existed, each with a known-correct expression.
- The agent was designed to emit an explicit `UNKNOWN` atom and a `NEED_INFO` status rather
  than guess when it could not parse part of a statement.
- Determinism and correctness on already-known inputs were hard requirements.
- Domain details are anonymised; figures are representative.

## Evidence

- Agent accuracy on the golden set plateaued around ~97%; further fine-tuning did not close
  the remaining gap reliably.
- Failure modes observed on near-duplicate inputs:
  - an expression returned with a **missing condition** (incomplete tree)
  - an **`UNKNOWN` atom** for a phrase previously mapped correctly
- Identical input submitted on different occasions could return different expressions.
- Every request incurred an LLM round-trip — latency and per-token cost — even for
  statements the system had already solved correctly.

## Investigation

1. **Is the model under-tuned?** Further fine-tuning against the golden set did not move
   accuracy reliably past ~97%. The residual error behaved like variance intrinsic to a
   probabilistic model, not a missing instruction. Ruled out "tune harder" as a path to
   full correctness.
2. **Are the failing inputs genuinely novel?** No — drift occurred on statements nearly
   identical to golden-set entries. Ruled in *re-deriving already-known answers* as the main
   source of both risk and waste.
3. **Does every request need the model?** No — only inputs never seen before require
   reasoning; known inputs already have a single correct answer established at authoring
   time. Ruled in a deterministic lookup as the primary path.
4. **Can a verified answer be trusted as data?** Yes, as long as the statement's meaning and
   the target schema do not change. Ruled in write-time verification with runtime reuse.

## Intervention

Inverted the control flow. A verified map from **normalised statement → expression** became
the authoritative source of truth, and the LLM became a fallback:

```text
Input arrives → normalise → look it up in the verified map
  ├─ HIT  → return the frozen, verified expression (deterministic, instant, no model call)
  └─ MISS → call the LLM → verify the result → write it back → future requests HIT
```

This targets the likely cause directly: drift came from asking a probabilistic model to
re-solve problems that already had one verified answer. Known inputs are now served from the
map, and model output is promoted into the map only after verification. Results containing
`UNKNOWN` or `NEED_INFO` go to a human review queue instead of being promoted.

## Result

- Verified inputs became **deterministic**: same normalised statement → same expression.
- The verified set is **correct by construction**: each entry is confirmed once by a human at
  write time instead of being re-derived at runtime.
- Latency for verified inputs dropped from an LLM round-trip to an in-process lookup.
- Model calls, and their cost, fall as recurring traffic is absorbed into the map; the cost
  of each unique input is paid once.
- Uncertainty: the guarantee covers only the verified set. Unseen inputs still depend on
  the model (~97% baseline) until reviewed, and overall benefit depends on how much real
  traffic the map covers.

## Trade-offs and limits

- Requires a **verification and write-back step**. A human or trusted check must confirm new
  entries before they become authoritative — this adds process, not just code.
- The map is only as correct as its verification. A bad entry becomes a permanent,
  deterministic error until corrected.
- Keys must be normalised (whitespace, casing, punctuation) or trivial variants miss.
  Paraphrases that normalisation does not collapse are still misses; widening matches with
  fuzzy or semantic similarity would reintroduce probabilistic behaviour at the lookup step.
- Entries must be re-verified or invalidated when the schema, atom vocabulary, or underlying
  eligibility rules change.
- Pays off only when inputs **recur** and outputs are **finite and structured**.
- It does not make the LLM more accurate; it makes the LLM *optional* for verified inputs.

## Related

- Problem family: [Something Is Nondeterministic](../README.md)
- Also relevant: [AI Is Wrong](../../ai-is-wrong/README.md)
- Pattern: [The Crystallization Pattern](../../../patterns/crystallization-pattern.md)

## Search terms

`nondeterminism` `llm-drift` `deterministic-lookup` `verified-map` `llm-fallback`
`write-time-verification` `probabilistic-to-deterministic` `schema-constrained-output`
`unknown-atom` `human-in-the-loop` `crystallization-pattern`

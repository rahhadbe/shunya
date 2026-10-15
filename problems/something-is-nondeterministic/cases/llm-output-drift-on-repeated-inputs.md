# Same input, different LLM answer — froze verified outputs into a lookup

## Problem

**In short:** an LLM produced the correct answer for an input, then later produced a
_different, wrong_ answer for the same or nearly the same input. Nothing in the system had
changed. The model had drifted from its own earlier response.

A production component used a tuned LLM to turn free-form, human-written text into a
structured output (fields, operators, values) that downstream code executed automatically.
The LLM was called on every request.

An illustrative example of what the drift looked like (not real data):

| Request   | Input                                | LLM output                                                                              |
| --------- | ------------------------------------ | --------------------------------------------------------------------------------------- |
| Monday    | "Orders over 100 shipped to the EU"  | `AND(order_total > 100, region = EU)` — correct                                         |
| Tuesday   | "orders above 100 shipped to EU"     | `order_total > 100` — a condition silently dropped                                      |
| Wednesday | "orders over 100 shipping for the EU" | `AND(order_total > 100, UNKNOWN)` — a phrase parsed correctly many times is now unknown |

Because the output drove a real automated decision, this was a correctness defect, not a
cosmetic one: the same input could lead to a different outcome on a different day, and no
error was raised. It became visible once volume grew and the same inputs began recurring at
scale.

## Context

- **Input:** effectively unbounded free text — people phrase the same thing in many ways.
- **Output:** finite and schema-constrained — a structured expression built from a fixed
  vocabulary.
- A golden set of ~450 unique inputs existed, each with a known-correct output.
- The model was designed to emit an explicit `UNKNOWN` element and a `NEED_INFO` status
  rather than guess when it could not parse part of an input.
- Determinism and correctness on already-known inputs were hard requirements.
- The problem is not specific to the domain: any system that asks an LLM to map free text
  to a fixed set of structured outputs on every request can show it.
- Domain details are anonymised; figures are representative.

## Evidence

- Model accuracy on the golden set plateaued around ~97%; further fine-tuning did not close
  the remaining gap reliably.
- Failure modes observed on near-duplicate inputs:
  - an output with a **missing condition** (incomplete structure)
  - an **`UNKNOWN` element** for a phrase previously mapped correctly
- Identical input submitted on different occasions could return different outputs.
- Every request paid for an LLM round-trip — latency and per-token cost — even for inputs
  the system had already solved correctly.

## Investigation

1. **Is the model under-tuned?** Further fine-tuning against the golden set did not move
   accuracy reliably past ~97%. The residual error behaved like variance intrinsic to a
   probabilistic model, not a missing instruction. Ruled out "tune harder" as a path to
   full correctness.
2. **Are the failing inputs genuinely new?** No — drift occurred on inputs nearly identical
   to golden-set entries. Ruled in _re-deriving already-known answers_ as the main source
   of both risk and waste.
3. **Does every request need the model?** No — only inputs never seen before require
   reasoning; known inputs already have a single correct answer. Ruled in a deterministic
   lookup as the primary path.
4. **Can a verified answer be trusted as data?** Yes, as long as the meaning of the input
   and the output schema do not change. Ruled in write-time verification with runtime reuse.

## Intervention

Inverted the control flow. A verified map from **normalised input → structured output**
became the authoritative source of truth, and the LLM became a fallback:

```text
Input arrives → normalise → look it up in the verified map
  ├─ HIT  → return the frozen, verified output (deterministic, instant, no model call)
  └─ MISS → call the LLM → verify the result → write it back → future requests HIT
```

This targets the likely cause directly: drift came from asking a probabilistic model to
re-solve problems that already had one verified answer. Known inputs are now served from the
map, and model output is promoted into the map only after verification. Results containing
`UNKNOWN` or `NEED_INFO` go to a human review queue instead of being promoted.

## Result

- Verified inputs became **deterministic**: same normalised input → same output.
- The verified set is **correct by construction**: each entry is confirmed once by a human at
  write time instead of being re-derived at runtime.
- Latency for verified inputs dropped from an LLM round-trip to an in-process lookup.
- Model calls, and their cost, fall as recurring traffic is absorbed into the map; the cost
  of each unique input is paid once.
- Uncertainty: the guarantee covers only the verified set. Unseen inputs still depend on the
  model (~97% baseline) until reviewed, and overall benefit depends on how much real traffic
  the map covers.

## Trade-offs and limits

- Requires a **verification and write-back step**. A human or trusted check must confirm new
  entries before they become authoritative — this adds process, not just code.
- The map is only as correct as its verification. A bad entry becomes a permanent,
  deterministic error until corrected.
- Keys must be normalised (whitespace, casing, punctuation) or trivial variants miss.
  Paraphrases that normalisation does not collapse are still misses; widening matches with
  fuzzy or semantic similarity would reintroduce probabilistic behaviour at the lookup step.
- Entries must be re-verified or invalidated when the output schema, its vocabulary, or the
  underlying business rules change.
- Pays off only when inputs **recur** and outputs are **finite and structured**.
- It does not make the LLM more accurate; it makes the LLM _optional_ for verified inputs.

## Related

- Problem family: [Something Is Nondeterministic](../README.md)
- Also relevant: [AI Is Wrong](../../ai-is-wrong/README.md)
- Pattern: [The Crystallization Pattern](../../../patterns/crystallization-pattern/crystallization-pattern.md)

## Search terms

`nondeterminism` `llm-drift` `same-input-different-output` `llm-consistency`
`text-to-structure` `deterministic-lookup` `verified-map` `llm-fallback`
`write-time-verification` `probabilistic-to-deterministic` `schema-constrained-output`
`human-in-the-loop` `crystallization-pattern`

# The Crystallization Pattern

## Intent

Make a probabilistic decision repeatable by serving verified answers from an authoritative
lookup, and invoking the probabilistic solver (typically an LLM) only for inputs not yet
verified: discover once, verify, freeze, reuse.

## The idea

### The problem it targets

Many tasks map **unbounded free-form input** to a **finite, structured output**: a category,
a queue, a route, a tool to call, a schema object, a logic tree. People phrase the same
thing in many ways, so a model is useful for understanding the input. But the set of valid
answers is small and well defined.

When a model is called on every request, three things happen:

1. **The decision becomes probabilistic.** The same input can produce a slightly different
   output on a different call — a dropped field, a different category, an `UNKNOWN` for
   something that was understood yesterday. Tuning reduces this variance but rarely removes
   it.
2. **Solved problems are solved again.** Once an input has a correct answer, asking the model
   again adds nothing except another chance to be wrong.
3. **Cost and latency scale with traffic, not with novelty.** You pay the full model price
   for the thousandth occurrence of an input exactly as for the first.

### The core move

Crystallization separates two activities that are usually mixed together:

- **Discovering** the answer for an input that has never been seen — this needs reasoning,
  and a model is good at it.
- **Serving** the answer for an input that has already been solved and verified — this needs
  only a lookup.

The verified answers live in an authoritative map (input → output). The model is moved
behind that map and only handles what the map does not yet know:

```text
Input arrives → normalise → look it up in a verified map (input → output)
  ├─ HIT  → return the frozen answer (deterministic, fast, verified)
  └─ MISS → call the solver → verify the result → write it back → future requests HIT
```

Over time, recurring inputs move from the slow, variable path to the fast, fixed path. The
name describes that change: from fluid (probabilistic output) to solid (a fixed, verified
answer).

### How it works, step by step

1. **Normalise the input.** Turn the raw input into a stable key: trim whitespace, fix
   casing, remove punctuation noise, and canonicalise obvious variants such as dates or
   numbers. Normalisation must itself be deterministic; it decides what counts as "the same
   input".
2. **Look it up.** An exact match on the normalised key returns the stored answer. No model
   is called, so the same key always gives the same answer.
3. **Solve on a miss.** Call the model with a schema-constrained output. Include an explicit
   "I don't know" state (for example `UNKNOWN` or `NEED_INFO`) so the model can decline
   instead of guessing.
4. **Verify.** Check the proposed answer before it is trusted: human review, a rule-based
   validator, a comparison with a golden set, or a combination. Anything containing an
   "I don't know" state goes to review rather than into the map.
5. **Promote.** Write the verified answer into the map with metadata (who verified it, when,
   and for which schema or rule version). From now on this input is a hit.
6. **Govern.** Treat the map as a source of truth: review changes, keep an audit trail, and
   re-verify or invalidate entries when the schema, the output vocabulary, or the underlying
   rules change.

A map entry typically holds:

```yaml
key: "invoice shows wrong amount" # normalised input
example_input: "My invoice shows the WRONG amount!!"
output: { queue: billing, sub_queue: disputes }
verified_by: reviewer-or-check-id
verified_at: 2026-01-15
schema_version: 3
```

### Design decisions you must make

| Decision                          | Options                                                                                  | Consideration                                             |
| --------------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| What is the key?                  | exact text, normalised text, extracted fields                                            | Wider keys raise the hit rate but must stay deterministic |
| How is an answer verified?        | human review, rule validator, golden-set comparison                                      | The map is only as correct as this step                   |
| What does a caller get on a miss? | the unverified model answer (flagged), a pending / `NEED_INFO` result, or a safe default | Depends on how harmful a wrong answer is before review    |
| How is the map seeded?            | empty, a golden set, historical decisions                                                | Seeding with known-good data gives hits from day one      |
| When are entries invalidated?     | on schema change, rule change, or an expiry review                                       | Stale entries are deterministic errors                    |

### A generic example: routing support tickets

A support system routes incoming tickets to one of 12 queues. Customers write in free text.
A model reads each ticket and picks the queue.

**Before.** Every ticket goes to the model. "My invoice shows the wrong amount" is routed to
`billing/disputes` most of the time, but occasionally to `billing/general` or `account`.
Every routing decision costs a model call, and agents complain that similar tickets land in
different queues.

**After crystallization:**

| Moment | Input                                                                        | What happens                                                                                                                                               |
| ------ | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Day 1  | "My invoice shows the wrong amount"                                          | Miss → model proposes `billing/disputes` → a team lead confirms → entry written                                                                            |
| Day 2  | "my invoice has the extra amount which is wrong"                             | Different wording, so a different key → miss → model proposes `billing/disputes` → confirmed → second entry written, mapped to the same `billing/disputes` |
| Day 3  | "my invoice shows the WRONG amount!!"                                        | Normalises to the Day 1 key → hit → `billing/disputes`, no model call                                                                                      |
| Day 4  | "Can you explain the refund policy for annual plans and also my login fails" | Model returns `NEED_INFO` (two intents) → sent to manual triage, not written to the map                                                                    |
| Later  | Billing queues are merged                                                    | Entries tagged with the old schema version are re-verified or removed                                                                                      |

Over weeks, the common phrasings are all hits. The model is called mainly for genuinely new
wording or new kinds of request. Routing for known tickets is identical every time, and the
number of model calls tracks how much _new_ wording arrives, not total ticket volume.

This example is illustrative; the case under [Evidence](#evidence) is the real application.

### What it is not

| Similar idea                                 | Difference                                                                                                                                                                                     |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cache                                        | A cache holds a disposable copy of whatever the model returned, usually unverified, and expires it. Here the stored entry is verified, authoritative, and kept until deliberately invalidated. |
| Semantic or similarity cache                 | Matching by embedding similarity returns answers for inputs that were never verified, which reintroduces probabilistic behaviour at the lookup step.                                           |
| Fine-tuning                                  | Fine-tuning makes the model better on average; it does not guarantee a specific input always gets a specific answer. Crystallization gives that guarantee for verified inputs.                 |
| Hand-written rules engine                    | Rules are written up front by people. Here the entries are discovered by the model and confirmed by people, so the map grows from real traffic.                                                |
| Few-shot examples or retrieval in the prompt | Examples steer the model but the model still decides. In crystallization a hit never reaches the model.                                                                                        |

### Key principles

- **A verified answer is a fact you own, not an oracle to re-consult.** Facts belong in a
  lookup, not behind a per-call model with built-in variance.
- **The model is a fallback, not the decision-maker** for anything already verified.
- **Correctness is asserted at write time, not gambled at runtime.**
- **"I don't know" is a valid output.** It routes work to people instead of becoming a
  silent error.
- **The map is a governed artifact**, with the same care as code or reference data.

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

- [Same input, different LLM answer — froze verified outputs into a lookup](../../problems/something-is-nondeterministic/cases/llm-output-drift-on-repeated-inputs.md)
  — an LLM converting free text to structured output gave different answers for inputs it
  had already solved and plateaued near 97% on a known set; a verified input → output map
  with LLM fallback made verified inputs deterministic, with latency and model cost falling
  as traffic recurred.

Currently supported by one case. Independent cases from other contexts (for example ticket
classification, request routing, or tool selection) would strengthen it.

## Related

- [Diagrams](diagrams.md) — architecture, request sequence, design decisions, comparison
- [Something Is Nondeterministic](../../problems/something-is-nondeterministic/README.md)
- [AI Is Wrong](../../problems/ai-is-wrong/README.md)

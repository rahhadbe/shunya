# SHUNYA

> A shared library of real engineering problems and the reasoning used to solve them.

SHUNYA is for the part of engineering that is usually lost after an incident is closed:

- what went wrong
- what engineers observed
- how they narrowed the problem
- what they changed
- what the change cost or made harder

There is no application code here. A contribution should preserve a useful way to think, not a technology tutorial.

## The simple model

Every contribution starts as a **case**: one real problem, in one context, with an honest outcome.

```text
case
  -> repeated learning
  -> pattern
  -> curated guide
  -> possible framework elsewhere
```

- **Case**: what happened, how it was investigated, and what was learned.
- **Pattern**: a solution idea that has worked in more than one meaningful context.
- **Guide**: a practical investigation path that connects several cases and patterns.

Do not start by proposing a pattern, guide, or framework. Start with the case. Maintainers can promote proven learning into the other forms.

Early patterns may be published as **seed patterns** when they are clearly labeled as unproven and invite supporting cases. They are prompts for investigation, not established advice.

## Where things live

```text
problems/
  <problem-family>/
    README.md              # The family: symptoms, questions, related cases
    cases/
      <case-name>.md       # A real engineering experience

patterns/
  <pattern-name>.md        # A reusable idea, linked to supporting cases

guides/
  <guide-name>.md          # A step-by-step path across cases and patterns

templates/
  case.md                  # Start here
  pattern.md               # Used when promoting repeated learning
  guide.md                 # Used for curated investigation paths
```

Problem families are deliberately broad: `something-is-slow`, `things-are-competing`, and `data-is-stale` are easier to discover than product or vendor names.

## A good case is small

A useful first contribution can be short. It only needs to answer:

1. What was the problem and what did it affect?
2. What evidence led you toward the cause?
3. What did you try, and why?
4. What changed after the intervention?
5. What trade-off or limitation should the next engineer know?

Use the [case template](templates/case.md). Keep proprietary details anonymous and replace sensitive values with representative ones.

## Example

```text
Problem: Orders occasionally timed out under load.

Evidence: Database wait time rose while application CPU stayed normal.

Investigation: We found transactions holding a lock during an external call.

Change: We moved the call before the short write transaction.

Result: Lock waits and timeouts fell.

Trade-off: The retry path now needs idempotency.
```

That case may later support a pattern such as "reduce lock duration." It does not need to claim that it is universally correct.

## Principles

- Problems before technologies.
- Evidence before advice.
- Reasoning before prescription.
- Trade-offs are part of the answer.
- One focused lesson beats a large, generic document.

Read the [contribution guide](CONTRIBUTING.md), browse the [problem taxonomy](docs/TAXONOMY.md), or see the [templates](templates/).

# Contributing to SHUNYA

The default contribution is a **case**. It records a real engineering problem and the reasoning used to address it.

You do not need to prove that your solution is a universal best practice. You do need to be clear about the evidence, result, and limits.

## Add a case

1. Find the closest problem family in `problems/`.
2. Create `problems/<family>/cases/<short-descriptive-name>.md` from [templates/case.md](templates/case.md).
3. Link the case from that family's `README.md`.
4. Open a pull request with one focused lesson.

If no family fits, propose a new one in the pull request. Choose a problem name, not a technology name.

Good: `things-are-competing`, `data-is-stale`, `change-is-dangerous`

Avoid: `postgresql-problems`, `aws-patterns`, `redis-guide`

## Promote learning

Promotion is optional and usually happens after more than one case supports the same lesson.

- Create a **pattern** when a solution idea applies across contexts and its trade-offs are understood. Use [templates/pattern.md](templates/pattern.md) and link supporting cases.
- Create a **guide** when an engineer needs a repeatable investigation path through several cases or patterns. Use [templates/guide.md](templates/guide.md).
- Build a framework outside SHUNYA only when the knowledge has become stable enough to deserve code or tooling. Link back to the cases and patterns that justified it.

## What belongs here

- incident learnings that can be safely shared
- diagnostic questions that changed the investigation
- failed approaches and why they failed
- measurements, experiments, or representative evidence
- reusable reasoning and explicit trade-offs

## What does not belong here

- code implementations or vendor tutorials
- generic "best practices" without context
- unverified claims
- sensitive customer, security, or operational details
- a solution presented as the only valid answer

## Pull request checklist

- The title describes the problem or lesson.
- The case explains the context without exposing sensitive information.
- The investigation contains evidence, not only conclusions.
- The result and trade-offs are documented.
- Related cases, patterns, or guides are linked when they exist.

AI assistance is welcome, but the contributor is responsible for the accuracy, safety, and originality of the entry.

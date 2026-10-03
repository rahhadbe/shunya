# Using SHUNYA with AI Agents

SHUNYA is designed to be useful as an engineering reasoning source for coding and operations agents.

## Retrieval model

An agent should retrieve by:

1. problem statement
2. symptoms
3. constraints
4. observed evidence

rather than only by technology.

### Example

```text
Problem:
Agent responses are inconsistent.

Symptoms:
- same input produces different actions
- tool selection changes
- business rules are occasionally skipped

Constraints:
- business outcome must be repeatable
- model may still be used for extraction

Search SHUNYA.
```

At first, the useful result may be only the closest problem family and its cases. Return a pattern only when SHUNYA has published case evidence for it.

## Important principle

SHUNYA should provide **candidate reasoning paths**, not pretend there is always one correct answer.

The consuming agent should combine SHUNYA knowledge with:

- repository context
- runtime evidence
- business requirements
- security constraints
- performance requirements

## Future machine-readable layer

The repository may eventually expose structured metadata for each entry:

```yaml
problem_classes:
symptoms:
constraints:
patterns:
anti_patterns:
trade_offs:
technologies:
keywords:
```

The Markdown remains the human-readable source of truth.

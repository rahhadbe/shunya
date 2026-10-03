# AI Is Taking The Wrong Action

Use this family when an AI system selects, authorizes, or executes an action that is unsafe, unsuitable, or outside its intended authority.

## What engineers notice

- the system chooses a valid tool for the wrong task
- an action bypasses a required approval or business rule
- similar inputs lead to materially different actions

## Questions to ask first

1. What action was proposed, and what authority should have limited it?
2. Was the error in interpretation, tool selection, validation, or execution?
3. Which actions need deterministic checks or human confirmation?

## Related patterns

- [Constrained Tool Selection](../../patterns/ai/constrained-tool-selection.md)
- [Validate Before Execute](../../patterns/ai/validate-before-execute.md)

## Cases

Add a real incident or investigation under `cases/` using the [case template](../../templates/case.md). Link it here with a one-line lesson.

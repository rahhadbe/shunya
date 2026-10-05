# Independent Checker Agreement

## Intent

Check AI output reliably when every available checker is imperfect. A failure is declared only when independent checkers agree. No checker that is biased toward "this looks fine" can approve on its own. Disagreement gets an explicit "not sure" result instead of a guess.

## The idea

Teams often check AI output with other imperfect tools: OCR to read text in an image, a second model to grade an answer, or a classifier to spot unsafe content. Each checker has blind spots, and some are biased in a dangerous direction. A spell-correcting reader "sees" correct words, and a model asked to grade output tends to read what *should* be there.

Relying on one checker forces a bad choice. A strict checker produces many false alarms, and a lenient one lets real failures through. This pattern combines several checkers under three rules:

```text
Run independent checkers on the same output
  ├─ two or more report the same problem → FAIL
  ├─ none report a problem               → PASS
  └─ they disagree                       → look closer (re-check with more effort)
        ├─ now confirmed → FAIL
        └─ still split   → UNVERIFIED → a person or a safe fallback decides
```

1. **Agreement to fail.** Checkers that fail for *different* reasons rarely make the same mistake, so a problem reported by two of them in the same place is very likely real. This removes most single-checker false alarms.
2. **Veto, not pardon.** A checker whose bias points toward approval, such as an AI model or an autocorrecting reader, may add or confirm a failure. It must never overrule the others to pass something.
3. **An explicit third state.** "Unverified" is a real outcome with its own path, such as human review, a deterministic fallback, or a retry. It never silently becomes "pass".

Independence matters more than the number of checkers. Two copies of the same engine, or two models from the same family, share blind spots and will agree on the same mistakes.

## Use it when

- A missed failure is expensive: wrong prices, wrong legal text, unsafe actions.
- You have at least two checkers that fail in *different* ways, for example a rule-based tool and a model, or two different engines.
- There is somewhere for "unverified" to go: a reviewer, a fallback, or a retry with a stronger method.
- You can label a test set, including deliberately planted failures, to measure catches and false alarms.

## Do not use it when

- A single deterministic check can decide exactly. For example, compare structured fields directly instead of reading them back with OCR.
- Your checkers are not truly independent: same engine, same model family, same training data. Agreement then confirms shared mistakes.
- Nobody can absorb the "unverified" volume, and the system has no safe default.
- A false alarm costs more than a missed failure. This pattern deliberately leans toward caution.

## Trade-offs

- **Improves:** fewer missed failures than a lenient single checker, fewer false alarms than a strict one, and no silent approvals by a biased checker.
- **Costs:** running several checkers costs more and takes longer; re-checking disagreements adds time; the "unverified" queue needs an owner.
- **Needs care:** "the same problem" must be defined precisely (same type of error, same location). Without that, two checkers can "agree" while describing different things.
- **Hides nothing, fixes nothing:** the pattern makes the decision more trustworthy. It does not make the generator any better.

## Evidence

- [The text checker fixed the typos it was supposed to catch](../problems/ai-is-wrong/cases/text-checker-hides-typos.md) - checking exact text in generated images. Requiring two readers to agree caught 15 of 15 bad-text images, and no reader could pass text alone. Single readers either missed failures or raised more false alarms.

Currently supported by one case. Independent cases from other contexts would strengthen it. Examples: using several graders to judge model answers, combining moderation classifiers, or cross-checking extracted document fields.

## Related

- [AI Is Wrong](../problems/ai-is-wrong/README.md)

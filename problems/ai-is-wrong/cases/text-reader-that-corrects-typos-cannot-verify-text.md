# A text reader that corrects typos cannot verify exact text

## Problem

An automated quality gate had to confirm that generated images contained an exact required string, such as an offer or a price. A wrong character ships a wrong ad, so a missed typo is the costly failure. One of the readers used to check the text quietly corrected spelling, so it could report a misspelled image as correct.

## Context

- A pipeline generated display ads from a product photo and a short required text, and then evaluated each image before a human approved it.
- Text was checked by reading the image and comparing the result with the required string. Character error rate had to be zero, and prices and percentages had to match exactly.
- Three readers were available: two general-purpose OCR engines and a vision-language model asked to transcribe the text.
- The evaluator was measured on 20 natural outputs plus 42 deliberately planted failures with labels known by construction (text: 6 typos, 3 missing, 2 stray), alongside a blind human labeller. 52 labelled items assessed the text dimension.

## Evidence

- One OCR engine has language correction on by default. With it on, a planted typo ("Prpmavera") and a planted stray `*` were read back as clean text, so those planted failures would have passed.
- Each reader alone, scored against the labels (positive class = fail):

  | Reader setup | Precision | Recall |
  | --- | --- | --- |
  | OCR only | 0.750 | 0.800 |
  | Vision model read-back only | 0.706 | 1.000 |
  | Two readers must agree (shipped) | 0.789 | 1.000 |

- A single OCR engine produced false alarms on stylised type. For example, it read a bold "4" as "A" at every scale and preprocessing tried, and it reported short noise tokens as stray text.

## Investigation

1. **Can any one reader be trusted to say "this text is correct"?** No. A reader that applies a language model or spell correction is biased toward real words, which is the wrong direction for catching typos. The correction setting had to be turned off and tested against planted typos.
2. **Can a single reader decide "this text is wrong"?** Not reliably. The OCR-only false alarms came from stylised fonts and noise, and one engine was alone in misreading a specific glyph. The vision model alone caught every failure but raised more false alarms.
3. **Do the readers fail in the same way?** No. The OCR engines and the vision model made different mistakes, which made agreement a useful signal.
4. **What should happen when the readers disagree?** Neither a pass nor a fail is safe, so the image needs a third state.

## Intervention

- Turned off language correction in every OCR reader and added planted typos as regression tests.
- A text failure requires two independent readers to report the same error at the same position.
- A difference that only one reader sees triggers a re-read of the text region at higher resolution with both engines. If it is still unconfirmed, the image is marked `unverified`, never passed.
- The vision model cannot pass text by itself. It can only add or confirm a failure ("veto, not pardon").
- `unverified` routes the image to repair or human review, and the system can draw the exact text deterministically if every other attempt fails.

## Result

- Recall stayed at 1.00. All 11 planted text failures (typos, missing text, stray text) were caught.
- Precision rose from 0.750 (OCR only) and 0.706 (vision only) to 0.789.
- The 4 remaining false alarms were all on items whose human labels were disputed. With those items excluded (48 items), precision and recall were both 1.00.
- The precision target of 0.85 was not met against the labels as recorded.

## Trade-offs and limits

- `unverified` adds work. Someone, or something, has to resolve those images, so throughput depends on how often the readers disagree.
- Higher-resolution re-reads add latency and compute to every disagreement.
- The second OCR engine was available on only one operating system. Elsewhere the agreement rule ran with one engine plus the vision model, which makes it weaker.
- "Agreement" needs a precise definition: same error type at the same position. One engine that misread a glyph at every scale forced a rule that an engine "sees" an error only if all of its reads show it.
- The evidence is small: 52 labelled items, one human labeller, and planted failures that may be easier than natural ones.
- The vision model came from the same family as the image generator, so the readers may share blind spots.

## Related

- Problem family: [AI Is Wrong](../README.md)

## Search terms

`ocr` `exact text` `language correction` `autocorrect` `typo` `vision model judge` `evaluator` `ensemble agreement` `generated images` `quality gate`

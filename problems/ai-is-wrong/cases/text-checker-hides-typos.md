# The text checker fixed the typos it was supposed to catch

## Problem

A system generated advertising images that had to contain an exact piece of text, such as "Primavera", "30% OFF" or "CHF 24.90". Image models often misspell text, so every image was checked automatically before a person approved it.

The check worked by **reading the text back out of the image** and comparing it with the text that was asked for. One of the tools doing the reading quietly corrected spelling as it read. When an image said "Prpmavera", the tool reported "Primavera". The checker then compared "Primavera" with "Primavera", saw a match, and passed an image with a typo in it.

A misspelled price or offer is the most expensive mistake this system can make, because the ad goes out with wrong information. The checker was failing in exactly that case.

## Context

- **What was being checked:** generated images that must show one short required text exactly, letter for letter and digit for digit.
- **The tools that read text from an image ("readers"):**
  - two OCR engines (optical character recognition: software that turns the pixels of letters into text), and
  - a vision-language model (an AI model that can look at an image and answer questions about it), asked to transcribe the text it sees.
- **How the checker was tested:** the team built a labelled test set of 52 images. Some were normal outputs. Others had failures planted on purpose, such as a typo inserted into correct text, text removed, or extra text added. Because the planted failures were made deliberately, the correct answer for each was known. A person also labelled every image as pass or fail without seeing the checker's verdict.
- **Two measures of a checker:**
  - **Caught:** of the images that really had bad text, how many the checker flagged. Missing one means a bad ad ships.
  - **False alarms:** images the checker flagged that were actually fine. Each one wastes a repair attempt or a reviewer's time.

## Evidence

- One OCR engine has spelling correction turned on by default. With it on, a planted typo ("Prpmavera" instead of "Primavera") and a planted stray `*` were both read as clean text. Those two broken images would have passed.
- With correction switched off, each reader used alone still had problems. On the 15 images with genuinely bad text:

  | Who decides | Bad images caught | False alarms |
  | --- | --- | --- |
  | OCR engines alone | 12 of 15 | 4 |
  | Vision model alone | all 12 it could assess | 5 |
  | Two readers must agree (final design) | 15 of 15 | 4 |

- The OCR engines raised false alarms on stylised fonts. One engine read a bold "4" as an "A" at every size and setting tried, and both picked up short specks of noise as "extra text".
- The vision model missed nothing it assessed, but it flagged more good images than any other setup. It also tends to read what a word *should* say, the same bias that caused the original problem.

## Investigation

1. **Can any single reader be trusted to say "the text is correct"?** No. A reader that corrects spelling leans toward real words, which is exactly the wrong direction when the job is to find misspellings. That ruled out letting any one reader approve text by itself.
2. **Can a single reader be trusted to say "the text is wrong"?** Not reliably. Each reader had its own blind spots: the OCR engines with stylised type and noise, the vision model with overcautious flags. Trusting either one alone meant either missing failures or raising many false alarms.
3. **Do the readers make the same mistakes?** No. The OCR engines and the vision model failed on different images for different reasons. When two different readers report the *same* error in the same place, it is very likely real.
4. **What should happen when the readers disagree?** Neither "pass" nor "fail" is safe. The system needed a third answer: "not sure".

## Intervention

The rule became: **a failure needs agreement, and no reader can approve on its own.**

```text
Read the text with every reader (spelling correction off)
  ├─ two readers report the same error  → FAIL       (send to repair)
  ├─ no reader reports any error        → PASS
  └─ only one reader reports an error   → re-read that area at higher resolution
        ├─ now confirmed by a second reader → FAIL
        └─ still only one reader            → UNVERIFIED (send to a person, never auto-pass)
```

- Spelling correction was switched off in every OCR engine, and the planted-typo images became permanent regression tests.
- The vision model can add or confirm a failure, but it can never overrule the others to pass an image. In short: it can veto, but it cannot pardon.
- If the text still fails after repair attempts, the system draws the exact text onto the image with ordinary code, so a shipped ad never depends on the model spelling correctly.

## Result

- Every bad-text image in the test set was caught: 15 of 15, including all 11 planted text failures.
- False alarms fell from 5 (vision model alone) to 4.
- All 4 remaining false alarms were images whose human label was later disputed. In 3 of them, the image model had drawn a "/" from the prompt into the ad ("Only / AED 49"). In the fourth, it had drawn a stray quote mark. The checker failed these images and the human passed them, so the checker was arguably right. When those 4 were set aside, the checker agreed with the human on every remaining image.
- The goal of no more than 15% false alarms among flagged images was **not met**: as labelled, 4 of 19 flagged images were false alarms, which is 21%.

## Trade-offs and limits

- **"Not sure" costs time.** Unverified images go to a person, so the more often readers disagree, the more manual review is needed.
- **Re-reading costs time and compute.** Every disagreement triggers extra reads at higher resolution.
- **The second OCR engine only ran on one operating system.** Elsewhere the rule ran with one OCR engine plus the vision model, which gives fewer independent opinions.
- **"Agree" has to be defined precisely.** The rule only works if two readers report the same kind of error at the same position. One engine misread the same letter every time, so the definition had to require an engine to see an error in *all* of its reads before it counts.
- **The evidence is small.** It rests on 52 labelled images, one human labeller, and planted failures that may be easier to spot than natural ones.
- **Shared blind spots are possible.** The vision model came from the same model family as the image generator, so both may make similar mistakes.

## Related

- Problem family: [AI Is Wrong](../README.md)
- Pattern: [Independent Checker Agreement](../../../patterns/independent-checker-agreement.md)

## Search terms

`ocr` `autocorrect` `spelling-correction` `typo` `exact-text` `text-in-images` `ai-evaluator` `vision-model-judge` `checker-agreement` `false-alarms` `unverified-state`

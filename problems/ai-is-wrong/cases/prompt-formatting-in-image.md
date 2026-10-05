# The prompt's formatting ended up in the image

## Problem

A system asked an image model to create advertisements that showed an exact headline, such as "Summer Sale" or "Only AED 49". To show the model which words were the headline, the prompt wrapped them in marker characters, the angle quotes « ». It used a "/" to show where a line should break.

The model did not treat these characters as instructions. It drew them. The first generated ads showed «Summer Sale» with the quote marks printed on the image. A later version of the prompt produced an ad that read "Only / AED 49".

The automated checker that should have caught this passed the images with a perfect score.

## Context

- **Where the text came from:** the headline was typed by a user and placed in the prompt. Wrapping user text in markers is a common way to keep it separate from the instructions around it.
- **How text was checked:** after generation, the system read the text back out of the image with OCR (optical character recognition: software that turns pixels of letters into text) and compared it with the requested headline. Any difference failed the image.
- **Why it mattered:** an ad with stray symbols around the price or offer looks broken and cannot ship.
- **When it was found:** on the first run against real briefs. Both images generated for the first brief had the « » drawn around the headline.

## Evidence

- Both images generated for the first brief showed the « » markers around the headline. The checker scored both as a perfect match.
- After the first fix, a different marker leaked. The prompt used "/" to mark a line break, and one of 20 images in a later run read "Only / AED 49".
- A human reviewer also passed the "/" image during labelling. The leaked character was easy to miss for people too.

## Investigation

1. **Why did the model draw the markers?** An image model treats everything in the prompt as a possible description of the picture. Characters placed next to the headline look like part of the headline. Nothing in the prompt told the model the markers were not to be drawn.
2. **Why did the checker pass a visibly wrong image?** For two separate reasons, each harmless alone:
   - **Clean-up:** before comparing, the checker normalised the text it read, folding fancy quotes into plain ones and then removing punctuation. That step deleted the drawn markers.
   - **Matching only the expected part:** the checker found the stretch of read text that best matched the headline and scored only that. Anything extra around it, including the markers, was ignored.
3. **Was this specific to « »?** No. The "/" leak showed that *any* formatting character placed next to the text could be drawn. The fix had to remove formatting from the text itself, not swap one marker for another.
4. **Could the markers be replaced with different characters?** That was considered and rejected. A substitute marker could be drawn just as easily. Changing the user's own punctuation to avoid a clash would make the model draw the wrong character in headlines that legitimately contain quotes.

## Intervention

**In the prompt:** the headline no longer carries any formatting.

- The whole headline goes on one line of its own, with nothing added around it.
- A separate sentence says, in words, how many lines the headline should be shown on. No symbol marks the line break.
- An explicit closing line marks the end ("End of headline copy.").
- The model is told not to add quotation marks, brackets or punctuation that are not in the headline.

**In the checker:**

- Clean-up now keeps quotes, brackets and similar marks whenever the requested headline does not contain them, so a drawn marker shows up as a difference instead of being deleted.
- Extra content read in the headline area now counts as an error, not just mismatches inside the matched part. To avoid failing on OCR noise, an extra counts only when two separate reads agree, the read is confident, and it has at least two letters or digits. A drawn marker always counts.
- New tests fail an image that has drawn markers or extra words, and pass one with only noise.

## Result

- Re-checked with the fixed checker, both original images failed. The checker reported the « » as "not in the required text".
- After the prompt change, a re-run of the same brief produced 4 of 4 images without markers, and all 4 passed the text check.
- After the line-break fix, none of 16 new, previously unseen briefs had a "/" drawn into the image.

## Trade-offs and limits

- **User-text separation moved elsewhere.** Markers had helped keep user text apart from instructions. Without them, that protection depends on other safeguards: validating the user's text before it reaches the model, and drawing suspicious text directly onto the image instead of sending it to the model.
- **A stricter checker needs noise rules.** Counting extras catches leaked markers, but it also risks failing on stray OCR noise. The thresholds (two agreeing reads, confidence, minimum length) were tuned for this system and may not transfer.
- **Known gaps remain.** A repeated word ("Sale Sale") is not counted as extra. A single extra character is tolerated unless it is a marker. Text at the very edge of the headline area is ignored.
- **The fix is prompt-specific.** It removes the markers this system used. A different prompt layout could leak different characters, so the checker, not the prompt, is the lasting protection.

## Related

- Problem family: [AI Is Wrong](../README.md)

## Search terms

`prompt-formatting` `delimiters` `prompt-leakage` `text-in-images` `image-generation` `ocr` `text-normalisation` `evaluator` `false-pass` `exact-text`

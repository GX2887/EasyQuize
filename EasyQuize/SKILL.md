---
name: paper-comprehension-quiz
description: >-
  Generate an interactive multiple-choice quiz that tests whether a reader truly
  understood a research paper. Use this skill whenever the user provides a PDF (or
  link/text) of a research paper, article, or technical report and wants to check
  comprehension — phrasings like "quiz me on this paper", "test my understanding",
  "did I get this paper", "make comprehension questions", "check if I understood
  this", or "make a quiz from this PDF". Trigger even if the user just drops in a
  paper PDF and asks to be tested, without naming the word "quiz".
---

# Paper Comprehension Quiz

This skill turns a research paper into a self-contained, interactive multiple-choice
quiz that a reader takes to confirm they actually understood the paper — not just
skimmed it. The reader clicks answers, submits, and immediately sees their score,
a per-section breakdown, and an explanation for every question.

The goal is an honest comprehension check, so the quiz must cover **every major
section of the paper** at a **core-facts** level: the central claim, the method,
the key results, and the main takeaways of each section. A reader who genuinely
read the paper should score well; a reader who only skimmed should not.

## Workflow

### 1. Get the paper's text

The user will provide a PDF (most common), a link, or pasted text.

For a PDF, extract the full text. The companion `pdf` skill is the reliable way to
do this — read its SKILL.md and use it to pull text. A quick path that usually works:

```bash
pip install pdfplumber --break-system-packages -q
python3 -c "
import pdfplumber, sys
with pdfplumber.open('PAPER.pdf') as pdf:
    print('\n'.join((p.extract_text() or '') for p in pdf.pages))
" > paper_text.txt
```

If the PDF is scanned (no extractable text), use the `pdf` skill's OCR path.

### 2. Map the paper's structure

Before writing any questions, read the text and list the paper's sections in order.
Most papers follow some version of: Abstract, Introduction/Motivation, Related Work,
Method/Approach, Experiments/Results, Discussion, Limitations, Conclusion. Use the
paper's *actual* section headings — they are your coverage checklist.

This matters because the user specifically wants **every chapter/section covered**.
A quiz that asks five questions all about the results section is not a real
comprehension check.

### 3. Write the questions

Aim for **one to two questions per major section**, scaling to the paper — a short
4-page paper might yield 6–8 questions; a long one, 12–15. Each question targets a
**core fact** a careful reader would retain:

- What problem does the paper address, and why does it matter?
- What is the key idea of the proposed method/approach?
- What was the main experimental setup or dataset?
- What is the headline result (the number or finding that matters)?
- What limitation or caveat do the authors acknowledge?
- What do the authors conclude or recommend?

Question-writing rules that keep the quiz fair and discriminating:

- Stick to **core facts stated in the paper**. Don't ask about trivia (a citation
  number, an author's affiliation) or require outside knowledge.
- Write **4 options** per question. The correct answer must be unambiguously
  supported by the text.
- Make the **distractors plausible** — common misreadings, plausible-sounding
  alternatives, or the right idea applied to the wrong section. Avoid joke options
  or obviously-wrong throwaways; they let a skimmer guess correctly.
- Keep stems self-contained and answerable without trick wording.
- Each `explanation` should point to *where in the paper* the answer comes from and
  why it's right, so the quiz teaches as it grades.

### 4. Build the interactive HTML quiz

Use the bundled template at `assets/quiz_template.html`. It already handles
answer selection, scoring, the per-section breakdown, and revealing explanations —
you only inject content. Do two replacements:

1. Replace `__PAPER_TITLE__` (appears twice) with the paper's title.
2. Replace `__QUESTIONS_JSON__` with a JSON array of question objects.

Each question object has this shape:

```json
{
  "section": "Method",
  "stem": "What is the central mechanism the authors propose to reduce inference cost?",
  "options": [
    "Caching attention keys and values across decoding steps",
    "Pruning low-magnitude weights after training",
    "Distilling the model into a smaller student network",
    "Quantizing activations to 8-bit integers"
  ],
  "answer": 0,
  "explanation": "Section 3 introduces a KV-cache reuse scheme; the other options are real techniques but not what this paper proposes."
}
```

`answer` is the 0-based index of the correct option. `section` should match one of
the paper's actual sections so the breakdown is meaningful.

The cleanest way to inject is a tiny script so the JSON stays valid:

```bash
python3 - <<'PY'
import json
questions = [ ... ]   # the list you wrote
html = open('assets/quiz_template.html', encoding='utf-8').read()
html = html.replace('__PAPER_TITLE__', 'Your Paper Title')
html = html.replace('__QUESTIONS_JSON__', json.dumps(questions, ensure_ascii=False))
open('<paper-name>-quiz.html', 'w', encoding='utf-8').write(html)
PY
```

Save the result as `<paper-name>-quiz.html` in the user's folder and present it so
they can open and take it. The file is fully self-contained — no internet needed.

### 5. Hand it off

Tell the user the quiz is ready, how many questions it has, and that it covers each
section. Offer to adjust difficulty, add more questions, or regenerate if they want
a fresh set.

## Quick quality check before delivering

- Does every major section of the paper appear in at least one question's `section`?
- Is each correct answer actually stated in the paper (no hallucinated specifics —
  especially numbers, which you should copy from the text, not invent)?
- Are the distractors plausible enough that a skimmer couldn't ace it by elimination?
- Does the HTML open and score correctly? (Open it or sanity-check the JSON is valid.)

<img width="1821" height="864" alt="logo" src="https://github.com/user-attachments/assets/02fa2622-7eff-49ae-912c-8efdc95890c6" />

# Paper Comprehension Quiz

A [Claude](https://claude.com) skill that turns a research paper into an interactive
multiple-choice quiz, so a reader can check whether they *actually* understood the
paper instead of just skimming it.

Drop in a PDF and the skill extracts the text, maps the paper's sections, writes
core-fact questions covering **every section**, and renders a self-contained HTML
quiz with instant scoring, a per-section breakdown, and an explanation for every
answer.

## What it does

- **Reads any research paper** — works from a PDF (text or scanned/OCR), link, or pasted text.
- **Covers the whole paper** — one to two questions per major section (Intro, Method,
  Results, Discussion, Limitations, Conclusion…), so a skimmer can't ace it on one section.
- **Tests core understanding** — questions target the central claim, method, key
  results, and takeaways, with plausible distractors drawn from common misreadings.
- **Interactive & offline** — outputs a single `*-quiz.html` file. Click answers,
  submit, and immediately see your score and where your gaps are. No internet required.

## Installing

**In Claude Cowork / Claude Desktop:** download `EasyQuiz.skill` and
open it, or use the **Save skill** button when it's presented to you.

**Manually:** copy the `EasyQuiz/` folder into your Claude skills
directory so that `SKILL.md` sits at its root.

## Using it

Attach a paper and ask, for example:

> "Quiz me on this paper."
> "Test whether I understood this PDF."
> "Make comprehension questions from this."

You'll get a `<paper-name>-quiz.html` file. Open it in any browser, answer the
questions, and submit. The per-section breakdown is the real signal — if you ace
Results but miss Method, you skimmed.

## How it's built

The skill is an instruction file (`SKILL.md`) plus a reusable HTML template
(`assets/quiz_template.html`). The template handles selection, scoring, and
explanations; Claude only injects the paper title and a JSON list of questions:

```json
{
  "section": "Method",
  "stem": "What is the central mechanism the authors propose?",
  "options": ["KV-cache reuse", "Weight pruning", "Distillation", "Quantization"],
  "answer": 0,
  "explanation": "Section 3 introduces a KV-cache reuse scheme."
}
```

## Repository layout

```
paper-comprehension-quiz/
├── SKILL.md                   # the skill: how Claude builds the quiz
├── assets/
│   └── quiz_template.html     # self-contained interactive quiz template
├── README.md
└── LICENSE
```

## License

MIT — see [LICENSE](LICENSE).

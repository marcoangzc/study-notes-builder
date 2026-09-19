---
name: study-notes-builder
description: Turn lecture slides, PDFs, textbook chapters or course notes into gap-filling study notes with worked examples, verified code/calculations, slide-error warnings and practice questions with full answers. Use this whenever the user uploads course material (slides, lecture PDFs, chapters, handouts) and asks for notes, revision material, a chapter summary, exam prep, or says the slides/lectures are hard to understand, unclear, or "not explained well" — even if they never say the word "notes". Also trigger for Chinese requests such as 做笔记, 整理笔记, 帮我复习, 讲义看不懂, 出练习题, 每一章做 note.
---

# Study Notes Builder

> 中文说明：这个 skill 会把老师的 slide / lecture 变成「补洞式」笔记：先完整读完材料，找出 slide 没讲清楚或讲错的地方，用例子和类比补上，代码和数字都实际验证过，最后附练习题和完整答案。

## Why this skill exists

Lecture slides are written to accompany a lecturer, not to teach on their own. Students who read them alone hit three walls: terms that are never explained, examples with the working missing, and statements that are vague or simply wrong. When a student hits a wall they tend to give up rather than ask. Good notes remove those walls **before** the student reaches them. The goal is never to restate the slides shorter; it is to explain what the slides skip.

## Workflow

Follow these steps in order. Do not start writing notes until steps 1–3 are done.

### 1. Scope the job (keep it quick)

Work out from the request and conversation: which files or chapters, the output language, and any format preference already stated. Ask at most one short question, and only if something essential is missing. If the user has a stored preference about language, teaching style or answer delivery, follow it silently.

Defaults when nothing is specified:
- One markdown file per chapter/lecture.
- Write in the language the user writes in. For Chinese-speaking users, explain in Chinese and keep all technical terms in English (Manglish/Malay-English mix is fine if the user uses it).
- Give answers directly, not "try it first" — unless the user asks for exercises without answers.

### 2. Read everything first

Read all the material for the requested chapters before writing anything. Reading only the chapter you are about to write causes wrong cross-references and duplicated explanations.

Check the real file format before assuming it is a normal PDF. Files named `.pdf` are sometimes zip archives of per-slide images plus text files; try `file <path>` and `unzip -l <path>` when `pdftotext` fails. Then use the right tool:

| Material | How to read |
|---|---|
| Text-based PDF | `pdftotext -layout` |
| Zip of slide images + txt | unzip, concatenate the txt files in numeric order |
| Images / scanned PDF | OCR (`tesseract`) if available, otherwise view the images |
| PPTX / DOCX | Use the pptx / docx skills to extract text |

Text extraction misses diagrams. When the surrounding text refers to a figure that matters (a graph, a tree, a memory layout, a circuit, a table shown as an image), open that image and look at it. This is not optional for figures that a worked example depends on: in one real case the DFS slide and the BFS slide used two *different* graphs, which only showed up by viewing the images. If you cannot inspect a figure, say so in the final message instead of guessing.

### 3. Find the gaps

While reading, keep a running list per chapter of anything a self-studying student would trip on:

- **Unexplained jargon** — a term used or defined in one line with no example.
- **Missing "why"** — the slide says what a thing is but not why it exists or why it matters.
- **Truncated or partial code** — a listing cut off at the slide boundary.
- **Questions posed on the slide with no answer** ("What is the output if…?").
- **Figures with no explanation** — diagrams the lecturer presumably talked through.
- **Internal contradictions** — slide 12 says one thing, slide 30 another.
- **Wrong or oversimplified statements** — verify these against reliable knowledge or by running code before you call them wrong.
- **Demo output that looks like a bug but is not** — usually because of state carried over between steps.
- **Concepts that quietly depend on an earlier chapter.**

This list drives the note. Sections that the slides already explain well can be short.

### 4. Verify before you write

Never put an unchecked claim into a note the student will memorise.

- Programming courses: run the code (Java, Python, etc.) and paste real outputs. Test the corrected versions of buggy slide code. If the runtime differs from the slide output (for example iteration order of a hash-based collection), say the difference is expected and why.
- Maths, algorithms, finance, physics: recompute every number in worked examples in code or by hand-checking, including the answers to your own practice questions.
- Theory subjects (law, biology, history, management): cross-check definitions against the source material, and label anything you add from outside the source as "extra / not in the slides" so the student knows what is exam-safe.
- If something cannot be verified, say so plainly in the note or the final message. A visible uncertainty is more useful than a confident error.

### 5. Write the notes

Use this structure for every chapter unless the user asks otherwise.

```
# <Course code> — Chapter N: <Title>
(one-line blockquote: what the slides get wrong or skip, and what this note adds)

## 0. One-sentence overview
   + a small map/table of the chapter's topics

## 1..k. Concept sections (one per topic, in slide order)
   - plain-language explanation
   - an analogy from everyday life when the concept is abstract
   - a worked example with a step-by-step trace table
   - "⚠️ common confusion" callouts where the gap list says students trip

## ⚠️ Where the slides mislead (only if there are real issues)
   table: slide number | what the slide says | more accurate understanding
   + one line on exam strategy: if a question quotes the slide, answer as the
     slide does, but the student should know the real picture

## Cheat sheet
   compact table of definitions / formulas / rules

## Practice (all four sections, answers included)
   A. MCQ (5)   B. Short answer (3)   C. Calculation / trace (3)   D. Thinking (2)
   Answers directly after the questions, with a short explanation for each.

## Links to other chapters
```

Writing principles:

- **Explain why, not only what.** If a rule exists, give the reason; students remember reasons.
- **Trace, don't assert.** Show the state after each step of an algorithm or calculation in a table.
- **One idea per section, in the order the lecture teaches it**, so the student can follow along with the slides open.
- **Keep the lecturer's terms and notation** (including odd ones like `←` for assignment) and note where they differ from common usage.
- **Analogies must be accurate.** A wrong analogy is worse than none. State where an analogy stops working if that matters.
- **Practice questions should test understanding**, mixing recall, working-through and "why" questions. Base at least some on the slide's own examples, since those are what the exam will resemble.
- **Diagrams**: use ASCII art in code fences inside markdown (it renders everywhere). Produce SVG only if the user asks, or if a diagram is too complex for ASCII and a file can be presented.
- **No HTML tags** such as `<details>` in markdown files; they may not render. Put answers in a plain "Answers" section.
- Length follows the gap list: a chapter with many gaps deserves a long note; a thin chapter should stay short. Do not pad.

### 6. Review before delivering

Re-read each note as a student who has never seen the course:

- Is every term defined at first use?
- Does every worked example have its working shown?
- Do the practice answers match a recomputation?
- Did you flag every slide problem you found, and only real ones?
- Are there any claims you did not verify and did not label?

### 7. Deliver honestly

- Save each chapter to `/mnt/user-data/outputs/` as its own `.md` file with a clear name (`<COURSE>_Ch<N>_<Topic>.md`).
- **Call `present_files` on the finished files.** Only tell the user a file is available after that call has actually happened. Never write "the file is below" if you have not attached it.
- Keep the closing message short: what each note covers, the most important slide errors you found, what you verified by running code, and any limitation (figures you could not inspect, parts you inferred, extras that are not from the slides).
- Offer the natural next step in one line (e.g. an interactive walkthrough of a hard chapter, more practice questions, diagrams as SVG), rather than a menu.

## Adapting to different subjects

**Programming / algorithms** — the pattern above applies as is. Run every snippet; include complexity where relevant; correct buggy slide code and explain the bug.

**Maths / statistics / physics** — put a full worked solution (each line justified) before the practice questions; check units and edge cases; recompute all numeric answers.

**Accounting / finance / economics** — use a small running example (one company, one dataset) across the chapter, show the journal entries or formulas line by line, and recompute totals.

**Law / business / humanities / biology** — replace code verification with cross-checking against the source text; use cases, scenarios or labelled diagrams as the "worked example"; practice questions lean toward application ("what happens if…") and compare-and-contrast.

**Language courses** — grammar patterns with contrasting correct/incorrect examples, and practice questions that require producing sentences.

## Optional adjustments the user may ask for

- **Exam-focused**: shorter explanations, bigger practice sets, a list of likely exam questions per chapter.
- **More practice**: extra calculation and trace questions, or past-paper style questions.
- **Flashcards / quiz**: use the quiz tool for interactive multiple-choice or flashcards from the note.
- **Interactive learning**: turn one hard chapter into a step-by-step conversation instead of a document.
- **One combined document** instead of one file per chapter.
- **Different language or level**: adjust the language mix and assumed background (for example SPM-level maths).

## Common pitfalls to avoid

- Starting to write after reading only one chapter.
- Repeating the slides in fewer words. If a section adds nothing the slide lacked, cut it.
- Declaring a slide "wrong" without checking. Many slide statements are simplifications rather than errors; label them as simplified.
- Copying long passages of the slides verbatim. Explain in your own words and use your own examples.
- Claiming a file, output or verification that did not actually happen.

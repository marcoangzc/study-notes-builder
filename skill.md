---
name: study-notes-builder
description: Turn lecture slides, PDFs, textbook chapters or course notes into standalone study notes that replace the slides — the student should be able to learn the subject from the note alone, having attended no lecture and never opened the original deck. Produces diagrams, worked examples, verified code/calculations, slide-error warnings, exam-answer formulations and practice questions with full answers. Use whenever the user uploads course material (slides, lecture PDFs, chapters, handouts) and asks for notes, revision material, a chapter summary, exam prep, or says the slides/lectures are hard to understand, unclear, or "not explained well" — even if they never say the word "notes". Also trigger for Chinese requests such as 做笔记, 整理笔记, 帮我复习, 讲义看不懂, 出练习题, 每一章做 note.
---

# Study Notes Builder

> 中文说明：这个 skill 产出的笔记要能**取代** slide，不是补充 slide。标准是：一个从没上过课、也不打算翻原本讲义的学生，只读这份笔记就能学会并且考得过。双语学生按「理解用母语、输出用考试语言」处理，不做逐句翻译。图能画就画。

## Why this skill exists

Lecture slides are written to accompany a lecturer, not to teach on their own. A student reading them alone hits terms that are never explained, examples with the working missing, figures the lecturer talked through, and statements that are vague or wrong. Most students hit a wall and stop rather than ask.

**The output standard is replacement-grade, not supplementary.** The test to apply to every finished note:

> A student who attended none of the lectures, will never open the original slide deck, and has only the stated prerequisites — can they learn this topic from the note alone and answer exam questions on it?

If any part of the note only makes sense with the slide open beside it, that part has failed. This is a higher bar than "summarise well" and it changes what goes in: prerequisites get stated, figures get redrawn rather than referenced, the lecturer's unspoken context gets written down, and nothing is left as a bare keyword.

**What a note honestly cannot replace**, and should never pretend to:
- The lecturer's own emphasis on what is examinable, and anything said only in class.
- Hands-on practice — running the code, doing the lab.
- Figures that could not be inspected (say so; see step 2).

Handle this by including a **slide index** (see step 5) so the student can find the original page for any section in seconds, and by naming these limits in the closing message. The goal is that the student *doesn't need* the slides, not that they *can't check* them.

## Workflow

Steps 1–4 come before any writing.

### 1. Scope the job (keep it quick)

Work out from the request and conversation: which files or chapters, the output language, the delivery format, and any preference already stated. Ask at most one short question, and only if something essential is missing. Follow stored preferences silently.

**Output language — decide explicitly, do not drift into the slides' language.** Course material is often in English even when the student thinks in another language, and it is easy to write the note in English just because the source is. Pick in this order:
1. What the user explicitly asks for ("用中文写", "write in Malay", "bilingual").
2. A stored language preference.
3. **The language of the user's own messages, not the language of the slides.**
4. If too short or ambiguous to tell, ask one question: "Which language should the notes be in?"

If the user's language differs from the exam's language, apply the bilingual model in the next section — this is the common case and does not need asking about.

**Format.** Default to a published HTML artifact when the session can publish one (see step 7). Only ask about format if the user hints at a constraint (printing, offline, submitting to someone).

### 2. Read everything first, figures included

Read all material for the requested chapters before writing anything. Reading only the chapter you are about to write causes wrong cross-references and duplicated explanations.

Check the real file format before assuming a normal PDF. Files named `.pdf` are sometimes zip archives of per-slide images plus text files; try `file <path>` and `unzip -l <path>` when `pdftotext` fails.

| Material | How to read |
|---|---|
| Text-based PDF | `pdftotext -layout` |
| Zip of slide images + txt | unzip, concatenate the txt files in numeric order |
| Images / scanned PDF | OCR (`tesseract`) if available, otherwise view the images |
| PPTX / DOCX | Use the pptx / docx skills to extract text |

**Viewing figures is mandatory, not optional.** Text extraction loses every diagram, and a replacement-grade note has to carry the diagram's content itself. After extracting text, find the slides whose text is missing or near-empty and **view those images**:

```bash
# list slides with little extractable text — these are the figure slides
for t in $(ls *.txt | sed 's/.txt//' | sort -n); do
  c=$(grep -v '^[0-9]*$' "$t.txt" | tr -d '[:space:]' | wc -c)
  [ "$c" -lt 25 ] && echo "figure-only slide: $t"
done
```

Also view any slide whose text refers to a figure that a worked example depends on (a graph, a tree, a memory layout, a circuit, a table rendered as an image), and any slide whose OCR text looks garbled — OCR flattens boxes-and-arrows diagrams into meaningless word soup, which reads as content and is not. In one real case the DFS and BFS slides used two *different* graphs, visible only by opening the images. In another, OCR turned a single Content-Provider diagram into what looked like three separate providers.

Section-divider slides with just a title can be skipped. If a figure genuinely cannot be inspected, say so in the note at that point and again in the closing message — never guess it.

**If a past exam paper, sample paper or assignment brief is present, read it.** It is the single highest-value file in the folder: it fixes the question style, the mark allocation, the verbs used ("examine", "justify", "differentiate"), and which topics actually carry marks. It reshapes the whole note — practice questions should imitate it, and step 5's exam-question map depends on it. Do not skip it because it isn't "content".

### 3. Find the gaps

While reading, keep a running list per chapter of anything a self-studying student would trip on:

- **Unexplained jargon** — a term used or defined in one line with no example.
- **Assumed prerequisites** — concepts from an earlier chapter or course the slide silently relies on.
- **Missing "why"** — what the thing is, but not why it exists or why it matters.
- **Truncated or partial code** — a listing cut off at the slide boundary.
- **Questions posed on the slide with no answer** ("What is the output if…?"). These are usually tutorial questions and often resemble exam questions — answer every one of them.
- **Figures with no explanation** — diagrams the lecturer talked through.
- **Internal contradictions** — slide 12 says one thing, slide 30 another.
- **Wrong or oversimplified statements** — verify before calling them wrong.
- **Absolute claims with real exceptions** — "X can never happen" where X sometimes can.
- **Demo output that looks like a bug but is not** — usually state carried over between steps.

This list drives the note's length and shape. Sections the slides already explain well stay short.

### 4. Verify before you write

Never put an unchecked claim into a note the student will memorise.

- **Programming** — run the code and paste real outputs. Test corrected versions of buggy slide code. If runtime differs from the slide's output (say, iteration order of a hash-based collection), explain why the difference is expected.
- **Maths, algorithms, finance, physics** — recompute every number, including the answers to your own practice questions.
- **Theory subjects** — cross-check definitions against the source, and label anything added from outside it as "extra / not in the slides" so the student knows what is exam-safe.
- If something cannot be verified, say so plainly. A visible uncertainty beats a confident error.

### 5. Write the notes

Structure for every chapter unless the user asks otherwise:

```
# <Course code> — Chapter N: <Title>
(one-line blockquote: what this note covers, what the slides get wrong or skip)

## 0. Overview
   - one-sentence summary + a map of the chapter's topics
   - 🎯 what the exam asks from this chapter (only if a past paper was read)
   - prerequisites: what you need to already know, with a one-line refresher each

## 1..k. Concept sections (one per topic, in slide order)
   - plain-language explanation, every term defined at first use
   - a diagram whenever the concept has structure (see "Visuals")
   - an analogy from everyday life when the concept is abstract
   - a worked example with a step-by-step trace table
   - 💬 答题句 (EN) — the exam-ready English formulation (see "Bilingual")
   - ⚠️ common-confusion callouts where the gap list says students trip

## ⚠️ Where the slides mislead (only if there are real issues)
   table: slide № | what the slide says | more accurate understanding
   + one line of exam strategy: answer as the slide does if quoted, but know the real picture

## Cheat sheet
   compact bilingual tables of definitions / formulas / rules

## Practice (all four sections, answers included)
   A. MCQ (5)  B. Short answer (3)  C. Application / calculation / trace (3)  D. Thinking (2)
   Written in the exam's language and imitating the past paper's style if one was read.
   Answers directly after the questions, each with a short explanation.

## Slide index
   table mapping each note section to the original slide numbers

## Links to other chapters
```

Writing principles:

- **Explain why, not only what.** Students remember reasons.
- **Define on first use, every time.** No forward references to terms the note hasn't introduced.
- **Trace, don't assert.** Show state after each step of an algorithm or calculation.
- **Carry the figure's content, don't cite it.** Never write "as shown on slide 14" — redraw or describe it so the note stands alone.
- **One idea per section, in the order the lecture teaches it**, so the note and the course stay aligned.
- **Keep the lecturer's terms and notation** (including odd ones like `←` for assignment) and note where they differ from common usage.
- **Analogies must be accurate.** A wrong analogy is worse than none; state where an analogy stops working if it matters.
- **Practice questions test understanding**, mixing recall, working-through and "why". Base some on the slide's own examples and, if a past paper exists, on its format.
- **Length follows the gap list.** A chapter with many gaps deserves a long note; a thin chapter stays short. Do not pad.

### 6. Review before delivering

Re-read each note as a student who has never seen the course **and does not have the slides**:

- Is every term defined at first use?
- Does any sentence require the slide to make sense? (Fix it.)
- Are the prerequisites stated, or silently assumed?
- Does every worked example show its working?
- Do the practice answers match a recomputation?
- Did you flag every slide problem you found, and only real ones?
- Are there unverified claims that are not labelled?
- Does every English formulation actually answer the question as posed?

### 7. Deliver

Pick the format in this order:

1. **Published HTML artifact (default).** Best for a replacement-grade note: real inline SVG diagrams, readable on a phone, one link the student keeps and can share, and it survives the conversation. Write a self-contained HTML file and publish it. Follow the session's publishing rules: theme-aware CSS variables, responsive layout, wide tables in their own `overflow-x: auto` container, no external assets.
2. **Markdown `.md` file** when publishing is unavailable or the user wants a file: one per chapter, named `<COURSE>_Ch<N>_<Topic>.md`, ASCII diagrams in code fences, no HTML tags such as `<details>` (they may not render).
3. **PDF or DOCX** only when the user wants to print, submit, or read offline — use the pdf / docx skills.

For any file, save to `/mnt/user-data/outputs/` and **call `present_files`**. Only tell the user a file is available after that call has happened.

Keep the closing message short: what each note covers, the most important slide errors found, what was verified by running code, and any limitation (figures that could not be inspected, parts inferred, extras not from the slides). Offer one natural next step, not a menu.

---

## Bilingual students: explain in the thinking language, answer in the graded language

Most bilingual students do not need the same content twice. They need each language for a **different job**:

| Job | Language |
|---|---|
| Understanding the concept | the language the student thinks in |
| Technical terms | **always the source/exam language** (that's how the question will be worded) |
| Answers they will write in the exam | **the exam's language** |

So the rule is not "translate everything". It is:

**Prose in the student's language. Terms untranslated. One exam-ready formulation per concept in the exam's language.**

Concretely, for a Chinese-speaking student on an English-taught course:

- **Body prose in Chinese.** Explanations, analogies, callouts, traces.
- **Never translate a technical term.** Write "Content Provider 管理对一个中央资料仓库的存取", not "内容提供者". The student must recognise the English term in the question paper. Give the Chinese gloss once, in the term table, then use English throughout.
- **After each major concept, one `💬 答题句 (EN)` line** — the sentence the student would actually write on the exam paper, in English, one or two sentences. This is *not* a translation of the paragraph above it; it is the scoring formulation, compressed. Example:

  > **💬 答题句 (EN)** — Each Android app is assigned a unique Linux user ID and runs in its own process with its own VM, so one app cannot access another app's files.

- **A term table per chapter**: `English | 中文 | 一句话说明`. This is the bridge, and it replaces every inline translation.
- **Practice questions and answers in the exam's language**, with a one-line Chinese note under each answer explaining why it scores. The student must practise *producing* English, not just reading it.
- **Cheat sheet bilingual**: term in English, explanation in Chinese.

This keeps the note roughly one language long while serving both jobs, and it trains the thing that actually loses marks — producing correct English under time pressure.

Adapt the pairing to the student: Malay–English, Tamil–English and so on work identically. If the user explicitly wants full parallel text in both languages, produce it, but say in one line that it roughly doubles the reading time for little gain over the model above.

---

## Visuals

Diagrams carry structure that prose cannot. Draw one whenever a concept has:

- **states and transitions** — lifecycles, protocols, state machines
- **layers or containment** — architectures, memory layouts, sandboxes, stacks
- **ordered flow** — request sequences, algorithm steps, message exchanges
- **two or more comparison dimensions** — a matrix beats three paragraphs
- **a figure from the slides** whose content the text extraction lost

Do **not** draw for definitions, flat lists, or anything already clear in one sentence. A decorative diagram costs the student attention and teaches nothing.

**How to produce them**, by format:

- **Published HTML artifact** — hand-write **inline SVG**. Colours from CSS variables so the diagram works in light and dark; `viewBox` with no fixed pixel width so it scales on a phone; real text elements (not paths) so it stays legible and selectable. This is the main reason the artifact is the default format.
- **Markdown file** — ASCII art in code fences. It renders everywhere and survives copy-paste. Keep it under ~70 characters wide so it doesn't wrap on a phone.
- **Never deliver the note through an inline chat visualiser.** Those widgets render in the conversation and vanish with it; the note has to be a file or a published page the student keeps.

Label every diagram with a caption saying what to notice in it. An unexplained diagram is the exact failure mode this skill exists to fix.

---

## Adapting to different subjects

**Programming / algorithms** — run every snippet; include complexity where relevant; correct buggy slide code and explain the bug. Diagrams for data structures and control flow.

**Maths / statistics / physics** — a full worked solution with each line justified before the practice questions; check units and edge cases; recompute all numeric answers.

**Accounting / finance / economics** — one running example (one company, one dataset) across the whole chapter; journal entries or formulas line by line; recompute totals.

**Law / business / humanities / biology** — replace code verification with cross-checking against the source text; use cases, scenarios or labelled diagrams as the "worked example"; practice questions lean toward application ("what happens if…") and compare-and-contrast.

**Language courses** — grammar patterns with contrasting correct/incorrect examples; practice questions that require producing sentences.

## Optional adjustments the user may ask for

- **Exam-focused**: shorter explanations, bigger practice sets, a list of likely exam questions per chapter.
- **One combined cheat sheet** across all chapters, organised by exam question type rather than by chapter.
- **More practice**: extra calculation and trace questions, or past-paper-style questions.
- **Flashcards / quiz**: use the quiz tool for interactive review from the note.
- **Interactive learning**: turn one hard chapter into a step-by-step conversation instead of a document.
- **Different level**: adjust the assumed background (for example SPM-level maths).

## Common pitfalls to avoid

- Writing after reading only one chapter.
- **Writing from OCR text without opening the figure slides.** The most common way a note ends up silently wrong.
- Leaving a section that only makes sense with the slide open — the whole point of the skill.
- Repeating the slides in fewer words. If a section adds nothing the slide lacked, cut it.
- Translating technical terms for a bilingual student, so they can't match the term to the exam paper.
- Declaring a slide "wrong" without checking. Many slide statements are simplifications, not errors — label them as simplified.
- Copying long passages of the slides verbatim. Explain in your own words with your own examples.
- Claiming a file, output or verification that did not actually happen.

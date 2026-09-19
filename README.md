# study-notes-builder

A Claude skill that turns lecture slides and course material into **gap-filling study notes**.

Slides are written to accompany a lecturer, so they often skip explanations, cut off code, or contain vague or wrong statements. This skill makes Claude read the whole course material first, find those gaps, fill them in, and check its work.

## What you get

For each chapter, one markdown note with:

- Plain-language explanations, analogies and step-by-step traces
- Warnings where the slides are misleading or wrong
- Code and calculations that were actually run and verified
- A cheat sheet
- Practice questions (MCQ, short answer, calculation/trace, thinking) with full answers

## Usage

1. Add `SKILL.md` to Claude (in Claude.ai, use the **Save skill** button on the file, or add it under your skills settings).
2. Upload your slides, PDFs or chapters.
3. Ask something like: *"Make study notes for chapters 1–5"* or *"These slides are hard to understand, can you explain them?"*

Works for programming, maths, accounting, law, biology and other subjects. Notes are written in the language you use.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The skill: triggers, workflow, note structure, subject adaptations |

## License

MIT (add a `LICENSE` file before publishing)

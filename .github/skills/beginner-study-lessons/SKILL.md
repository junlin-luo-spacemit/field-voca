---
name: beginner-study-lessons
description: Create, review, or improve beginner-friendly SpacemiT study lessons and companion quizzes, preserving technical accuracy, bilingual terminology, and continuity across sessions.
---

# Beginner study lessons and quizzes

Use this workflow when creating or revising the SpacemiT study materials, explaining them to a beginner, or building their companion quizzes.

## 0. What this material is for

This is a **technical foundation course for the writing team**, not a writing course. The learner's stated goal: build real understanding of SpacemiT's products, chips and software ecosystem. Writing is a later concern.

Organization is by **technical thread**, not by document library. Every module pulls from several of the six official doc libraries (docs-product, docs-chip, docs-buildroot, docs-bianbu, docs-ai, docs-tool). Never structure a module as "today we read one library". See `spacemit/study/ROADMAP.md`.

Do not restate the goal as "how to write documentation" or "what a writer must check". That framing was tried and rejected.

## 1. Establish context

1. Read `spacemit/study/README.md`, `spacemit/study/ROADMAP.md`, `spacemit/study/HANDOFF.md`, the relevant existing lessons and quizzes, and applicable repository guidance. The handoff is a snapshot, not proof of current status or learner understanding.
2. Identify the learner's stated goal and specific confusion. If the topic or scope is unclear, ask before writing. Do not infer mastery from a quiz score or completion.
3. Consult the relevant primary documentation where available. Record source location and applicable version, platform, date and test conditions when they affect a claim. If a source is unavailable, disclose that and mark claims requiring confirmation; do not invent support, specifications or benchmarks.

## 2. Write or review the lesson

- Define a small, practical learning objective and assumed prerequisites. Each lesson explains one thing well.
- Explain concepts in plain Chinese by default, unless the learner requests another language. Introduce important technical terms as Chinese plus English and expanded acronyms on first use; explain unfamiliar words rather than stacking jargon.
- Short paragraphs, one idea each. The learner has repeatedly asked for short paragraphs and clear layering over dense prose.
- Build from what it is, to a concrete example, to how related things differ, to why the design is like that. Do not assume intermediate knowledge.
- Prefer real document excerpts and concrete numbers over abstract analogy. When using analogy, mark its limits.
- Separate general principles from product-specific claims. Qualify compatibility, performance and support by model, hardware, software version and workload. Avoid absolute promises and unsupported figures.
- Do not require commands, configurations, detailed mathematics or exhaustive specifications unless the objective needs them. Include an applicable source and short self-check questions.

## 3. Design and deliver the quiz

- Align questions with the stated lesson objectives and material actually taught. Cover core distinctions and realistic decisions, not mere acronym recall.
- Use the repository's three-part quiz structure (`spacemit/study/ROADMAP.md`): (1) basic single-choice questions, (2) "spot the trap" questions whose options are real common misconceptions, (3) a per-concept self-assessment. The whole quiz must be answerable without writing any prose — most learners will not write.
- Keep self-assessment per concept, never one blanket "did you understand everything". Treat it as a record and a hint for what to revisit, never as evidence of mastery or a gate.
- Make each question unambiguous, with one defensible answer and plausible alternatives. Trap options must come from genuine misconceptions, not invented ones. Avoid trick questions and unsupported guarantees.
- Explain why the answer is correct and clarify the misconception behind a wrong choice. Link back to the relevant lesson section. Feedback should be encouraging and must not treat score as proof of understanding.
- **Read `spacemit/study/STYLE.md` first and follow it exactly.** It is the binding spec for lesson writing and quiz delivery. The summary below is a pointer, not a substitute.
- Every question carries: `q`, `opts`, `a`, `e`, `wrong` (per-option), `src` (to section), `rv` (2–3 sentences quoted verbatim from the lesson), `sai` (index of the self-assessment row it maps to). Never align questions to self-assessment rows by question number — use `sai`.
- The result view must show a **three-tier revisit list**: (1) wrong *and* marked "懂了" — cognitive blind spot, most dangerous; (2) wrong; (3) marked "没懂/有疑问". Re-sort it whenever self-assessment changes, not only on completion.
- Track per-question failure counts in `localStorage` (`<module>.history`). At 2+ failures on the same question, tell the learner **the explanation likely needs rewriting**, not that they should re-read. That distinction is the point: it separates a weak lesson from a weak learner.
- Provide a **copy-diagnosis button** producing a paste-ready summary (attempt number, wrong items with counts, self-assessment doubts, contradictions, items flagged for rewrite). Use `navigator.clipboard` with an `document.execCommand("copy")` fallback — clipboard is often blocked on `file://`.
- "重做" clears page state but **keeps** history counts; otherwise counting is meaningless.
- 12–15 questions per 3-lesson module, 4–5 per lesson, hard cap 15. Every self-assessment row needs at least one question; delete the row or add the question rather than leaving a mismatch.
- Deliver the quiz link and explain how to use the result view. Never claim the learner understood unless they demonstrate it. Do not claim you will follow up automatically — you only act when the learner sends a message; the copy button is the bridge.
- Deliver a link to the quiz and explain how to use it. Never claim the learner understood unless they demonstrate it.

## 4. Verify and hand off

1. Check technical claims and quiz answers against the lesson and cited sources. Check links and ensure changes do not assert unverified product support.
2. For interactive changes, test all of: initial rendering; correct and incorrect choices; duplicate-score prevention; completion totals and group breakdowns; the three-tier revisit ordering; revisit re-ordering after a self-assessment change; the 2x-failure rewrite warning; the copy-diagnosis output; and reset (page cleared, history retained). Record actual results; if a test cannot be performed, say so explicitly and note it in HANDOFF.md.
3. Update the study index and `spacemit/study/HANDOFF.md` when appropriate: record completed changes, unresolved questions, source limitations, verified versus unverified behavior, and next steps. Preserve existing user changes and avoid claiming more learner progress than the user reported.

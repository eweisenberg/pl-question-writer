---
name: pl-question-writer
description: >
  Use this skill for any PrairieLearn DSA1 question — coding (JUnit-graded Java) or non-coding
  (multiple choice, true/false, multiselect, matching, short answer, order blocks). Triggers:
  writing or editing question.html, server.py, Tests.java, info.json, or any PrairieLearn question.
---

# PrairieLearn DSA1 Question Writer

## Step 1 — Determine intent and get the QID

**Writing a new question:** Confirm the QID (the directory name PrairieLearn created, e.g.
`bstInsertRecursive`). If not provided, stop and say:
> "Please add the question through the PrairieLearn localhost web interface first, then tell me
> the QID (the directory name, not the human-readable title) so I know where to write the files."

**Editing an existing question:** Ask for the QID if not given, then read and modify only the
file(s) relevant to what needs to change:
- Question wording, display, or static answer choices → `question.html`
- Randomization logic or version content → `server.py`
- Test cases or point values → `tests/junit/Tests.java`
- Topic or tags → `info.json` (uuid is never touched)

Do not rewrite files that don't need to change.

---

## Step 2 — Clarify question type, randomization, and (for coding) point total

If the user hasn't specified the question type, ask:
> "What type of question? Multiple choice, True/False, Multiselect, Matching, Short answer, Order blocks, or Coding?"

If the user hasn't specified whether to randomize, ask:
> "Should this question be randomized (different version each attempt), or static (same every time)?"

Skip this question for: static coding questions (never randomized), matching (always controlled by pool size — ask how many statements to show), and multiselect (always controlled by pool size — ask how many answers to show).

**Difficulty tag:** The user decides the difficulty, but always suggest one based on the question:
- `"easy"` — single concept, straightforward recall or one-step application
- `"medium"` — applies one concept with some nuance, or combines two simple concepts
- `"hard"` — multi-step reasoning, tricky edge cases, or combines multiple concepts

Do not suggest or assign `"extreme"` to new questions — only `"easy"`, `"medium"`, or `"hard"`.

---

## Step 3 — Load the reference file for the confirmed type

Read **only** the file for the chosen type. Do not load files for other types.

| Question type | Reference file |
|---|---|
| Multiple choice | `references/question-types/multiple-choice.md` |
| True/False | `references/question-types/true-false.md` |
| Multiselect | `references/question-types/multiselect.md` |
| Matching | `references/question-types/matching.md` |
| Short answer | `references/question-types/short-answer.md` |
| Order blocks | `references/question-types/order-blocks.md` |
| Coding | `references/question-types/coding.md` |

---

## Step 4 — Write the files

Produce `question.html`, `server.py` (if needed), and for coding: `tests/junit/Tests.java`.
Update only `topic` and `tags` in `info.json` (see info.json rules below).
Present every file in a labeled code block.

**For coding questions only:** after presenting all files, also output a sample solution — a complete, correct `Solution.java` (or equivalent class file) that would pass all the tests. Label it clearly as "Sample Solution (for testing)" so the user can paste it in to verify the grader works before releasing the question.

---

## info.json Rules

**NEVER modify `uuid`.** It is a permanent PrairieLearn identifier. Only ever change `topic`
and `tags`. All other fields are either managed by PrairieLearn or handled separately.

If the question doesn't fit an existing topic or tag, **do not invent one** — offer 2–4 closest
options and let the user choose.

**Approved topics (exact, case-sensitive):**
`"Abstract"` · `"ArrayList/Vector"` · `"Arrays"` · `"Big O"` · `"Binary Search Trees"` ·
`"AVL/Red Black Trees"` · `"Classes"` · `"Collections Framework"` · `"Comparable/Comparator"` ·
`"Concurrency"` · `"Data Structures"` · `"Equals"` · `"Exceptions"` · `"Generics"` ·
`"Hash Tables"` · `"Heap Sort"` · `"Inheritance"` · `"Interface"` · `"LinkedList"` ·
`"Merge Sort"` · `"Objects/References"` · `"Polymorphism"` · `"Priority Queues/Heaps"` ·
`"Queue"` · `"Quick Sort"` · `"Radix Sort"` · `"Recursion"` · `"Searching/Sorting"` ·
`"Sets/Maps"` · `"Stack"` · `"Trees/Traversals"`

For non-coding questions, only reference Java and topics from this list in stems, answers, and distractors. If unsure about a concept — even as a distractor — ask the user first.

**Approved tags (exact, case-sensitive):**
- Type (pick one): `"coding"` · `"multiple choice"` · `"multiselect"` · `"matching"` · `"short answer"` · `"order blocks"`
- Modifier: `"randomized"` (include only when randomized)
- Difficulty (pick one): `"easy"` · `"medium"` · `"hard"` (do not use `"extreme"` for new questions)

**Coding questions** also need these added to info.json (not in the PrairieLearn scaffold):
```json
"gradingMethod": "External",
"externalGradingOptions": { "enabled": true, "image": "prairielearn/grader-java", "timeout": 20 },
"singleVariant": true
```


---

## Images

Images go in `clientFilesQuestion/` inside the question directory. Tell the user the exact path:
`questions/<QID>/clientFilesQuestion/<filename>.png`. Create the folder if it doesn't exist.
Embed with: `<pl-figure file-name="filename.png" inline="true"></pl-figure>`

**If the question needs a diagram, load the appropriate reference file and generate a filled-in snippet for the specific diagram needed (not a generic template):**

| Diagram type | Reference file |
|---|---|
| UML class diagram | `references/diagrams/uml.md` |
| Linked list | `references/diagrams/linked-list.md` |
| Binary tree / BST / AVL / Red-Black | `references/diagrams/tree.md` |

Present the filled-in snippet alongside the tool instructions from the reference file so the user can generate the image immediately.

---

## General Style Rules

- Use `<markdown>` for prose inside `<pl-question-panel>`
- Use `<pl-code language="java">` for code shown to students
- Use `&lt;` / `&gt;` for less than or greater than signs (e.g. generics)

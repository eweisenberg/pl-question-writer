# True/False Reference

A special case of multiple choice with `order="fixed"` so True always appears before False.
Can be static (one fixed statement) or randomized (pool of statements, one drawn per attempt).

## info.json tags
- Randomized: `["multiple choice", "randomized", "medium"]`
- Static: `["multiple choice", "medium"]` — no server.py; hardcode `correct="true"` or `correct="false"` directly on each `<pl-answer>` (see Example B)

---

## question.html (randomized)
```html
<pl-question-panel>
  <p>{{{params.question_prompt}}}</p>
</pl-question-panel>

<pl-multiple-choice answers-name="ans" order="fixed" hide-letter-keys="true">
  <pl-answer correct="{{{params.true_answer}}}">True</pl-answer>
  <pl-answer correct="{{{params.false_answer}}}">False</pl-answer>
</pl-multiple-choice>
```

**Important:** Use triple-braces `{{{...}}}` (not double) for boolean params so they are not
HTML-escaped. Double-braces would render `true` as a string, not a boolean.

---

## server.py
```python
import random

def generate(data):
    scenarios = [
        ("Statement A that is true.", True),
        ("Statement B that is false.", False),
        ("Statement C that is true.", True),
        ("Statement D that is false.", False),
        # Add more — aim for 8–15 statements for good coverage
    ]

    statement, answer = random.choice(scenarios)
    data["params"]["question_prompt"] = statement
    data["params"]["true_answer"] = answer
    data["params"]["false_answer"] = not answer
```

---

## Notes
- Provide a large pool (8–15 statements) to reduce repetition across attempts
- Mix roughly equal numbers of true and false statements in the pool
- Statements should be unambiguous — avoid "sometimes" or "usually" wording
- If showing a code snippet as the statement, set `question_prompt` to the HTML string
  with `<pl-code>` tags and use `{{{params.question_prompt}}}` (triple braces) in the template

---

## Example

### Example A: Randomized — Large Pool (treesTrue)

**Demonstrates:** Large pool of statements randomly sampled each attempt. Mix of true and false.

**server.py:**
```python
import random

def generate(data):
    scenarios = [
        ("A binary tree with n nodes always has exactly n edges.", False),
        ("In a complete binary tree, all levels are filled except possibly the last level, which is filled from left to right.", True),
        ("Every complete binary tree is also a full binary tree.", False),
        ("All perfect binary trees are also complete.", True),
        ("A binary search tree can have duplicate values if the implementation allows it.", True),
        ("The height of a perfect binary tree with n nodes is log2(n+1) - 1.", True),
        ("An in-order traversal of a BST always visits nodes in sorted order.", True),
        ("A full binary tree always has an odd number of nodes.", True),
    ]
    statement, answer = random.choice(scenarios)
    data["params"]["question_prompt"] = statement
    data["params"]["true_answer"] = answer
    data["params"]["false_answer"] = not answer
```

*(question.html is identical to the template at the top of this file)*

### Example B: Static — Pass by Value

**Demonstrates:** A single fixed statement where the answer never changes. No server.py needed. Use `order="fixed"` so True always appears first; hardcode `correct="true"` or `correct="false"` directly.

**info.json tags:** `["multiple choice", "medium"]`

```html
<pl-question-panel>
    <markdown>Java arguments are always "pass by value".</markdown>
</pl-question-panel>

<pl-multiple-choice answers-name="ans" order="fixed" hide-letter-keys="true">
    <pl-answer correct="true"><markdown>True</markdown></pl-answer>
    <pl-answer correct="false"><markdown>False</markdown></pl-answer>
</pl-multiple-choice>
```

**Note:** Static T/F is appropriate when the statement is a known, unambiguous fact that doesn't benefit from a pool (e.g., a single core concept definition). For anything with multiple plausible statements, prefer the randomized pool pattern (Example A).

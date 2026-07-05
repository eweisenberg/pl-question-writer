# Short Answer Reference

Student types a number or short text. Graded automatically against `data["correct_answers"]`.

## info.json tags
- Randomized: `["short answer", "randomized", "medium"]`
- Static: `["short answer", "medium"]`

---

## question.html — numeric answer
```html
<pl-question-panel>
    <markdown>Question stem referencing `{{params.someVar}}`.</markdown>
</pl-question-panel>

<pl-integer-input answers-name="ans" label="Answer ="></pl-integer-input>
```

For decimal answers use `<pl-number-input answers-name="ans" comparison="sigfig" digits="3">` instead.

## question.html — text answer
```html
<pl-string-input answers-name="ans" label="Answer:" ignore-case="true" remove-spaces="true"></pl-string-input>
```

---

## server.py
```python
import random

def generate(data):
    version = random.randint(1, 3)

    if version == 1:
        data["params"]["someVar"] = "value A"
        data["correct_answers"]["ans"] = 42       # integer
        # or: data["correct_answers"]["ans"] = "someText"  # for string input

    elif version == 2:
        data["params"]["someVar"] = "value B"
        data["correct_answers"]["ans"] = 7

    else:
        data["params"]["someVar"] = "value C"
        data["correct_answers"]["ans"] = 15
```

**Key:** correct answer goes in `data["correct_answers"]`, not `data["params"]`.

---

## Notes
- For static questions, set `data["correct_answers"]["ans"]` in `generate()` with no `random` call
- `ignore-case="true"` and `remove-spaces="true"` on `<pl-string-input>` prevent grading issues
- For numeric answers, prefer `<pl-integer-input>` for whole numbers; it shows a cleaner input
- Avoid open-ended text answers that require judgment — PrairieLearn does exact string matching
- Good use cases: count of nodes, height of tree, Big-O class (e.g. `"O(n)"`)

---

## Examples

### Example A: Randomized — Switch Statement Trace

**Demonstrates:** server.py computing the correct answer programmatically from the randomized params, rather than hardcoding it. The `performOperation` helper mirrors the Java switch logic in Python.

**question.html:**
```html
<pl-question-panel>
<markdown>What is the value of `x` after the following code executes?</markdown>
<pl-code language="java">int x = {{params.initial}};
switch (x) {
    case 1:
        {{params.case1}};
    case 2:
        {{params.case2}};
    case 3:
        {{params.case3}};
    case 4:
        {{params.case4}};
}</pl-code>
</pl-question-panel>

<pl-integer-input answers-name="ans" placeholder="Type answer here"></pl-integer-input>
```

**server.py:**
```python
import random

def generate(data):
    initial = random.randint(1, 3)
    data["params"]["initial"] = initial
    cases = ["x++", "x--", "x += 2", "x -= 2", "x *= 2", "x /= 2",
             "x = x * x", "x *= -1", "x = 0", "x = 1"]
    case1 = random.choice(cases)
    case2 = random.choice(cases)
    case3 = random.choice(cases)
    case4 = random.choice(cases)
    data["params"]["case1"] = case1
    data["params"]["case2"] = case2
    data["params"]["case3"] = case3
    data["params"]["case4"] = case4

    # Mirror the Java switch fall-through logic in Python to compute the answer
    x = initial
    if initial == 1:
        x = performOperation(x, case1)
    if initial <= 2:
        x = performOperation(x, case2)
    if initial <= 3:
        x = performOperation(x, case3)
    x = performOperation(x, case4)
    data["correct_answers"]["ans"] = x

def performOperation(x, op):
    match op:
        case "x++":      return x + 1
        case "x--":      return x - 1
        case "x += 2":   return x + 2
        case "x -= 2":   return x - 2
        case "x *= 2":   return x * 2
        case "x /= 2":   return int(x / 2)
        case "x = x * x": return x * x
        case "x *= -1":  return x * -1
        case "x = 0":    return 0
        case "x = 1":    return 1
```

**Key pattern:** When the correct answer depends on randomly chosen params, compute it in `server.py` using Python logic that mirrors the Java behavior exactly. Never hardcode answers for generated inputs.

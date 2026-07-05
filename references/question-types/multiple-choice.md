# Multiple Choice Reference

One correct answer, 3–4 distractors. PrairieLearn shuffles answer order automatically.

## info.json tags
- Randomized: `["multiple choice", "randomized", "medium"]`
- Static: `["multiple choice", "medium"]`

---

## Randomized — question.html
```html
<pl-question-panel>
    <markdown>Question stem. Use `{{params.variable}}` for randomized parts.</markdown>
    <!-- Optional: show code in stem -->
    <pl-code language="java">Your code here</pl-code>
</pl-question-panel>

<pl-multiple-choice answers-name="ans" hide-letter-keys="true">
    <pl-answer correct="true"><markdown>{{params.correct}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect1}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect2}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect3}}</markdown></pl-answer>
</pl-multiple-choice>
```

## Randomized — server.py
```python
import random

def generate(data):
    version = random.randint(1, 3)  # add more versions as needed (aim for 3–6)

    if version == 1:
        data["params"]["correct"] = "Correct answer"
        data["params"]["incorrect1"] = "Distractor A"
        data["params"]["incorrect2"] = "Distractor B"
        data["params"]["incorrect3"] = "Distractor C"
    # elif version == 2: ...
    # elif version == 3: ...
```

For nested randomization (outer = subtopic, inner = version within that subtopic):
```python
topic = random.choice(["insertion", "deletion", "search"])
if topic == "insertion":
    version = random.randint(1, 3)
    # set params per version
```

---

## Static — question.html (no server.py needed)
```html
<pl-question-panel>
    <markdown>Which of the following is true about X?</markdown>
</pl-question-panel>

<pl-multiple-choice answers-name="ans" hide-letter-keys="true">
    <pl-answer correct="true">Correct statement.</pl-answer>
    <pl-answer correct="false">Wrong statement A.</pl-answer>
    <pl-answer correct="false">Wrong statement B.</pl-answer>
    <pl-answer correct="false">Wrong statement C.</pl-answer>
</pl-multiple-choice>
```

---

## Notes
- `hide-letter-keys="true"` is always required (course standard)
- Use `<markdown>` inside `<pl-answer>` for inline code spans (`` `someCode` ``)
- For multi-line code in an answer choice, use `<pl-code language="java">` inside the answer
- Distractors should represent plausible misconceptions, not obviously wrong statements

---

## Examples

### Example A: Static — Code Analysis (vector-find-code)

**Demonstrates:** Show buggy code, ask what's wrong. `show-line-numbers="true"` helps for "what is wrong on line N" questions. No server.py needed.

```html
<pl-question-panel>
  <p>Consider the method below — the find method for a vector. Will it work correctly?</p>
  <pl-code language="java" show-line-numbers="true">
public int find(T data) {
    for(int i=0; i &lt; this.dataStorage.length; i++) {
        if(this.dataStorage[i] == data) {
            return i;
        }
    }
    return -1;
}
  </pl-code>
</pl-question-panel>

<pl-multiple-choice answers-name="cpu" hide-letter-keys="true">
  <pl-answer correct="false">This method works as intended.</pl-answer>
  <pl-answer correct="false">The loop indices are incorrect (runs too few times).</pl-answer>
  <pl-answer correct="false">The loop indices are incorrect (runs too many times).</pl-answer>
  <pl-answer correct="true">This method does not work because equality is checked incorrectly.</pl-answer>
  <pl-answer correct="false">This method will never return -1 even when it should.</pl-answer>
</pl-multiple-choice>
```

---

### Example B: Randomized — BST Insertions (BSTinsertions)

**Demonstrates:** Randomized stem that references a sequence of values; correct answer is always the degenerate (sorted/reverse-sorted) case.

**question.html:**
```html
<pl-question-panel>
    <markdown>Which of the following sets of values, when inserted into a Binary Search Tree
    in the order given, will result in the time complexity of all operations being O(n)?
    </markdown>
</pl-question-panel>

<pl-multiple-choice answers-name="ans" hide-letter-keys="true">
    <pl-answer correct="true"><markdown>{{params.correct}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect1}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect2}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect3}}</markdown></pl-answer>
</pl-multiple-choice>
```

**server.py:**
```python
import random

def generate(data):
    version = random.randint(1, 4)
    if version == 1:
        data["params"]["correct"] = "{5, 17, 390, 401, 437, 500, 999}"
        data["params"]["incorrect1"] = "{999, 500, 437, 401, 17, 5, 390}"
        data["params"]["incorrect2"] = "{401, 17, 5, 500, 390, 437, 999}"
        data["params"]["incorrect3"] = "{401, 17, 999, 5, 500, 390, 437}"
    # elif version == 2: different ascending/descending sequences...
```

---

### Example C: Randomized — Multiple Subtopics (AVLall)

**Demonstrates:** Outer random selects the *question type*, inner random selects the *version*. Each subtopic sets its own `numbers` (shown in the stem) and answer params independently. Use this pattern when one question entry covers several related subtopics.

**server.py:**
```python
import random

def generate(data):
    topic = random.choice(["height", "parent", "remove", "rotate"])

    if topic == "height":
        version = random.randint(1, 3)
        if version == 1:
            data["params"]["numbers"] = "50, 30, 70, 20, 10, 60, 65"
            data["params"]["correct"] = "2"
            data["params"]["incorrect1"] = "3"
            data["params"]["incorrect2"] = "4"
            data["params"]["incorrect3"] = "1"
        elif version == 2:
            data["params"]["numbers"] = "10, 20, 30"
            data["params"]["correct"] = "2"
            data["params"]["incorrect1"] = "0"
            data["params"]["incorrect2"] = "1"
            data["params"]["incorrect3"] = "3"
        # elif version == 3: ...

    # elif topic == "remove": different numbers + answers for this subtopic
```

**question.html:**
```html
<pl-question-panel>
    <markdown>Insert the following values into a BST in the order given: `{{params.numbers}}`.
    What is the height of the root of the resulting tree, assuming the height of a leaf is 0?</markdown>
</pl-question-panel>

<pl-multiple-choice answers-name="ans" hide-letter-keys="true">
    <pl-answer correct="true"><markdown>{{params.correct}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect1}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect2}}</markdown></pl-answer>
    <pl-answer><markdown>{{params.incorrect3}}</markdown></pl-answer>
</pl-multiple-choice>
```

Note: `params.numbers` is always set inside each `if/elif` version block — never rely on a param being set in an outer scope.

# Multiselect Reference

"Select all that apply." PrairieLearn draws a random subset of answers to display each attempt,
ensuring at least `min-correct` correct answers are shown.

## info.json tags
- Static (pool size = `number-answers`): `["multiselect", "medium"]`
- Randomized (pool size > `number-answers`): `["multiselect", "randomized", "medium"]`

A server.py is only needed if the question *stem* or *context* also needs to vary.

---

## question.html
```html
<pl-question-panel>
    <markdown>Which of the following statements about **X** are **true**? Select all that apply.</markdown>
</pl-question-panel>

<pl-checkbox answers-name="ans"
             hide-letter-keys="true"
             number-answers="4"
             min-correct="1"
             max-correct="4"
             partial-credit="true"
             partial-credit-method="EDC">
    <pl-answer correct="true"><markdown>True statement 1.</markdown></pl-answer>
    <pl-answer correct="true"><markdown>True statement 2.</markdown></pl-answer>
    <pl-answer correct="true"><markdown>True statement 3.</markdown></pl-answer>
    <pl-answer correct="false"><markdown>False statement A.</markdown></pl-answer>
    <pl-answer correct="false"><markdown>False statement B.</markdown></pl-answer>
    <pl-answer correct="false"><markdown>False statement C.</markdown></pl-answer>
</pl-checkbox>
```

**Attributes:**
- `number-answers` — how many choices to display per attempt (drawn from pool above)
- `min-correct` / `max-correct` — PrairieLearn guarantees this range of correct answers shown
- `partial-credit-method="EDC"` — course standard; do not change

---

## server.py (only if stem varies)
```python
import random

def generate(data):
    version = random.randint(1, 3)
    if version == 1:
        data["params"]["context"] = "an ArrayList"
    elif version == 2:
        data["params"]["context"] = "a LinkedList"
    else:
        data["params"]["context"] = "a Stack"
```

If the stem is static, no server.py is needed — the pool draw in `<pl-checkbox>` already
provides randomization.

---

## Notes
- Provide more answers in the pool than `number-answers` so the draw is meaningful (6+ total)
- Ensure true and false statements are roughly balanced in the pool
- Each statement should be independently true or false (not depend on another statement)
- Use `<markdown>` inside `<pl-answer>` for inline code; `<pl-code language="java">` for blocks

---

## Examples

### Example A: Static — Generics Wildcard (refuelAll)

**Demonstrates:** `<pl-code>` inside answer choices for code-as-answer questions; all answers shown (no pool draw needed when pool equals `number-answers`). No server.py.

```html
<pl-question-panel>
<markdown>Consider the classes `Vehicle` and `Truck`, where `Truck extends Vehicle`. Given the following code, which of the following method headers for the `refuelAll` method will compile? Select all that apply.</markdown>
<pl-code language="java">ArrayList&lt;Truck&gt; trucks = new ArrayList&lt;&gt;();
// Trucks added to list (not shown)
refuelAll(trucks);</pl-code>
</pl-question-panel>

<pl-checkbox answers-name="ans" hide-letter-keys="true" number-answers="5" partial-credit="true" partial-credit-method="EDC">
    <pl-answer correct="true"><pl-code language="java">public static void refuelAll(ArrayList&lt;Truck&gt; trucks)</pl-code></pl-answer>
    <pl-answer correct="true"><pl-code language="java">public static void refuelAll(ArrayList&lt;?&gt; trucks)</pl-code></pl-answer>
    <pl-answer correct="true"><pl-code language="java">public static void refuelAll(ArrayList&lt;? extends Vehicle&gt; trucks)</pl-code></pl-answer>
    <pl-answer correct="false"><pl-code language="java">public static void refuelAll(ArrayList&lt;Vehicle&gt; trucks)</pl-code></pl-answer>
    <pl-answer correct="false"><pl-code language="java">public static void refuelAll(ArrayList&lt;? super Vehicle&gt; trucks)</pl-code></pl-answer>
</pl-checkbox>
```

**info.json tags:** `["multiselect", "medium"]` — no `"randomized"` tag since all answers are always shown.

---

### Example B: Randomized — Generics Wildcard with Pool Draw

**Demonstrates:** When the pool is larger than `number-answers`, PrairieLearn draws a random subset each attempt.

Take Example A and add 2+ extra `<pl-answer>` choices (e.g. `List&lt;Truck&gt;` wrong, `List&lt;? extends Vehicle&gt;` correct) so the total pool exceeds `number-answers="5"`. PrairieLearn picks 5 per attempt.

**info.json tags:** `["multiselect", "randomized", "medium"]` — include `"randomized"` whenever pool size > `number-answers`.

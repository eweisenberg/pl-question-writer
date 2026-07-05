# Matching Reference

Students drag statements into the correct category. Can be static or randomized. Randomization requires no server.py — simply include more `<pl-statement>` entries than `number-statements`. PrairieLearn draws a different subset each attempt.

## info.json tags
- Static (all statements always shown): `["matching", "medium"]`
- Randomized (pool > `number-statements`, different subset each attempt): `["matching", "randomized", "medium"]`

---

## question.html
```html
<pl-question-panel>
    <markdown>Match each statement to the correct category.</markdown>
</pl-question-panel>

<pl-matching answers-name="ans" counter-type="full-text" number-statements="6">
    <pl-statement match="Category A"><markdown>Statement 1</markdown></pl-statement>
    <pl-statement match="Category B"><markdown>Statement 2</markdown></pl-statement>
    <pl-statement match="Category A"><markdown>Statement 3</markdown></pl-statement>
    <pl-statement match="Both"><markdown>Statement 4</markdown></pl-statement>
    <pl-statement match="Category B"><markdown>Statement 5</markdown></pl-statement>
    <pl-statement match="Both"><markdown>Statement 6</markdown></pl-statement>
    <!-- Add extra statements beyond number-statements for pool-based randomization -->
    <pl-statement match="Category A"><markdown>Extra statement 7</markdown></pl-statement>
</pl-matching>
```

**Attributes:**
- `counter-type="full-text"` — shows the full category name as the label (course standard)
- `number-statements="N"` — how many statements to show per attempt; set lower than total
  pool size to get randomization without server.py
- `match="CategoryName"` — must exactly match one of the category names used across all statements

---

## Notes
- No server.py is ever needed for matching — randomization is controlled entirely by pool size vs `number-statements`
- Categories are inferred automatically from the `match` values — no need to declare them
- Use `<markdown>` inside `<pl-statement>` for inline code spans
- Balance statements across categories so no single category dominates
- Typical patterns: "A / B / Both", "O(1) / O(log n) / O(n) / O(n²)", interface vs abstract class

---

## Example

### Example A: Static — Abstract Class vs Interface (abstractClassInterfaceMatching)

**Demonstrates:** Three-way categorization ("Abstract class" / "Interface" / "Both"); pool larger than `number-statements` for lightweight randomization without server.py.

```html
<pl-question-panel>
    <markdown>Match each statement to whether it applies to **abstract classes**, **interfaces**, or **both**.</markdown>
</pl-question-panel>

<pl-matching answers-name="ans" counter-type="full-text" number-statements="7">
    <pl-statement match="Abstract class"><markdown>Used with the `extends` keyword</markdown></pl-statement>
    <pl-statement match="Interface"><markdown>Used with the `implements` keyword</markdown></pl-statement>
    <pl-statement match="Abstract class"><markdown>You can extend/implement only one of these</markdown></pl-statement>
    <pl-statement match="Interface"><markdown>You can extend/implement more than one of these</markdown></pl-statement>
    <pl-statement match="Abstract class"><markdown>Can have instance variables</markdown></pl-statement>
    <pl-statement match="Interface"><markdown>Can only have static and final variables</markdown></pl-statement>
    <pl-statement match="Both"><markdown>You can declare a variable of this type</markdown></pl-statement>
    <pl-statement match="Both"><markdown>Can be used as a parameter type in a method</markdown></pl-statement>
    <pl-statement match="Abstract class"><markdown>Can have a constructor</markdown></pl-statement>
</pl-matching>
```

Pool has 9 statements; `number-statements="7"` means 7 are shown per attempt — lightweight randomization with no server.py.


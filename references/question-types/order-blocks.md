# Order Blocks Reference

Students drag code lines/blocks into the correct order. All `<pl-answer>` blocks are always declared in `server.py`, with question.html using a Mustache loop to render them.

**Display options (optional — only add `display-blocks` and `solution-placement` if the user asks for inline layout):**
```html
<!-- Default: blocks stacked vertically, answer pane on the right -->
<pl-order-blocks answers-name="ans" grading-method="ranking" partial-credit="lcs">

<!-- Inline layout with solution pane at the bottom (only if user requests) -->
<pl-order-blocks answers-name="ans" grading-method="ranking" partial-credit="lcs"
                 display-blocks="inline-wrap" solution-placement="bottom">
```

## info.json tags
- Static (fixed blocks every attempt): `["order blocks", "medium"]`
- Randomized (block content or ordering varies via server.py): `["order blocks", "randomized", "medium"]`

---

## question.html (always the same structure)
```html
<pl-question-panel>
    <markdown>Arrange the following lines to correctly implement **[method/algorithm]**.</markdown>
</pl-question-panel>

<pl-order-blocks answers-name="ans" grading-method="ranking" partial-credit="lcs">
    {{#params.blocks}}
    <pl-answer correct="{{correct}}" ranking="{{ranking}}"><markdown>`{{content}}`</markdown></pl-answer>
    {{/params.blocks}}
</pl-order-blocks>
```

**Attributes:**
- `grading-method="ranking"` + `partial-credit="lcs"` — awards partial credit for longest correct subsequence (course standard)
- The Mustache loop renders one `<pl-answer>` per item in `params.blocks`
- Distractors: include in the blocks list with `"correct": false` and omit `ranking` (or set to 0)

---

## server.py

**Static** (blocks are fixed every attempt):
```python
def generate(data):
    data["params"]["blocks"] = [
        {"correct": True,  "ranking": 1, "content": "first correct line;"},
        {"correct": True,  "ranking": 2, "content": "second correct line;"},
        {"correct": True,  "ranking": 3, "content": "    indented third line;"},
        {"correct": False, "ranking": 0, "content": "a distractor line;"},
        {"correct": False, "ranking": 0, "content": "another distractor;"},
    ]
```

**Randomized:** see the Example below for the full pattern.

---

## Notes
- Use `&lt;` and `&gt;` for angle brackets inside block content
- Include 2–3 plausible distractor lines representing common mistakes
- Common distractor types: off-by-one (`i < n` vs `i <= n`), wrong operator, swapped operands
- The Python sort logic for randomized questions must exactly mirror the Java logic

---

## Example: Randomized — Movie Sort Order (MovieComparator)

**Demonstrates:** server.py generates blocks and correct rankings dynamically; Mustache loop renders them.

**Image:** Generate `movie_uml.png` using the UML diagram tool (see SKILL.md) for the `Movie` class with fields `-genre: String`, `-title: String`, `-length: int` and its constructor/getters. Place in `questions/<QID>/clientFilesQuestion/movie_uml.png`.

PlantUML for this diagram:
```
@startuml
hide circle
skinparam classAttributeIconSize 0

class Movie {
    -genre: String
    -title: String
    -length: int
    +Movie(genre: String, title: String, length: int)
    +getGenre(): String
    +getTitle(): String
    +getLength(): int
}
@enduml
```

**question.html:**
```html
<pl-question-panel>
<markdown>Consider the UML class diagram for the `Movie` class (defined in `Movie.java`) and the `MovieComparator` class shown below:</markdown>
<pl-figure file-name="movie_uml.png" inline="true"></pl-figure>

<pl-code language="java">
{{params.movie_comparator_code}}
</pl-code>

<markdown>Arrange the following `Movie` objects in increasing order, assuming `MovieComparator` is used to compare each `Movie`. Put the first `Movie` in sorted order at the top.</markdown>
</pl-question-panel>

<pl-order-blocks answers-name="ans" grading-method="ranking" partial-credit="lcs">
    {{#params.blocks}}
    <pl-answer correct="{{correct}}" ranking="{{ranking}}"><markdown>`Movie("{{genre}}", "{{title}}", {{length}})`</markdown></pl-answer>
    {{/params.blocks}}
</pl-order-blocks>
```

**server.py:**
```python
import random
import functools

def generate(data):
    movies = [
        {"genre": "Drama",     "title": "Moonlight",    "length": 113},
        {"genre": "Action",    "title": "Speed",        "length": 113},
        {"genre": "Comedy",    "title": "Ted",          "length": 113},
        {"genre": "Thriller",  "title": "Die Hard",     "length": 131},
        {"genre": "Adventure", "title": "Jumanji",      "length": 131},
        {"genre": "Fantasy",   "title": "King Kong",    "length": 131},
        {"genre": "Romance",   "title": "Her",          "length": 142},
        {"genre": "Sci-Fi",    "title": "Matrix",       "length": 142},
        {"genre": "Animation", "title": "Nemo",         "length": 142},
        {"genre": "Epic",      "title": "Interstellar", "length": 169},
        {"genre": "Drama",     "title": "Parasite",     "length": 132},
        {"genre": "Action",    "title": "Inception",    "length": 148},
    ]

    selected_movies = random.sample(movies, 5)
    sort_length_desc = random.choice([True, False])
    sort_title_desc  = random.choice([True, False])

    length_comp_java = ("Integer.compare(m2.getLength(), m1.getLength())" if sort_length_desc
                        else "Integer.compare(m1.getLength(), m2.getLength())")
    title_comp_java  = ("m2.getTitle().compareTo(m1.getTitle())" if sort_title_desc
                        else "m1.getTitle().compareTo(m2.getTitle())")

    data["params"]["movie_comparator_code"] = f"""import java.util.Comparator;
public class MovieComparator implements Comparator<Movie> {{
    @Override
    public int compare(Movie m1, Movie m2) {{
        int compLength = {length_comp_java};
        if (compLength != 0) return compLength;
        return {title_comp_java};
    }}
}}"""

    def python_compare(m1, m2):
        diff = (m2["length"] - m1["length"]) if sort_length_desc else (m1["length"] - m2["length"])
        if diff != 0:
            return diff
        if sort_title_desc:
            return -1 if m2["title"] < m1["title"] else (1 if m2["title"] > m1["title"] else 0)
        else:
            return -1 if m1["title"] < m2["title"] else (1 if m1["title"] > m2["title"] else 0)

    selected_movies.sort(key=functools.cmp_to_key(python_compare))

    data["params"]["blocks"] = [
        {"correct": True, "ranking": i + 1,
         "genre": m["genre"], "title": m["title"], "length": m["length"]}
        for i, m in enumerate(selected_movies)
    ]
```

# Linked List Diagram

## Instructions
1. Go to https://dreampuf.github.io/GraphvizOnline
2. Paste the generated DOT code below into the editor.
3. Set the output format dropdown to **png** and click **Download**.
4. Place the file in `questions/<QID>/clientFilesQuestion/<filename>.png`
5. Embed in question.html: `<pl-figure file-name="<filename>.png" inline="true"></pl-figure>`

## Template — Singly Linked List
```
digraph G {
    rankdir = "LR";
    node [shape = "circle" fontname = "Arial"];
    1 -> 2 -> 3;
}
```

## Template — Doubly Linked List
```
digraph G {
    rankdir = "LR";
    node [shape = "circle" fontname = "Arial"];
    edge [dir = "both"];
    1 -> 2 -> 3;
}
```

## Notes
- Node labels can be any string; quote labels with spaces: `"head" -> 3 -> 7 -> null`
- To show a null terminator: add a node `null [shape=plaintext]` and connect to it
- When generating a diagram for a question, produce a filled-in DOT snippet (not the template) with the exact node values and structure from the question

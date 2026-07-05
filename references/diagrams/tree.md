# Binary Tree / BST / AVL / Red-Black Tree Diagram

## Instructions
1. Go to https://eweisenberg.github.io/binary-tree-visualizer/lab/index.html
2. Open `tree_visualizer.ipynb` and run the first (large) code cell.
3. Paste the generated Python snippet below into the second code cell and run it.
4. Run the third cell as-is to save the image.
5. Right-click `tree.png` in the file browser → **Download**.
6. Place the file in `questions/<QID>/clientFilesQuestion/<filename>.png`
7. Embed in question.html: `<pl-figure file-name="<filename>.png" inline="true"></pl-figure>`

> To reset the notebook to its original state, clear site data for `eweisenberg.github.io` in browser settings.

## Second cell template
```python
# Second arg sets node color: "red" or "black" for Red-Black trees; omit for default (white)
root = TreeNode(5)
root.left = TreeNode(2)
root.left.left = TreeNode(1)
root.left.right = TreeNode(3)
root.right = TreeNode(9)
```

## Third cell (run as-is)
```python
img = generate_png_image(root)
img.save("tree.png", "PNG")
```

## Notes
- Build the tree by setting `.left` and `.right` on each `TreeNode`
- For Red-Black trees, pass `"red"` or `"black"` as the second argument: `TreeNode(5, "red")`
- Null children do not need to be declared — omitting `.left` or `.right` leaves that child empty
- When generating a snippet for a question, produce a filled-in Python snippet (not the template) that builds the exact tree from the question

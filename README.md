# pl-question-writer

This is a Claude skill for writing and editing PrairieLearn questions for Data Structures and Algorithms 1 (CS 2100) at the University of Virginia. Given a description of the question, the agent will produce working code for `question.html`, `server.py`, and other files as necessary, following the conventions and style of existing questions.

## Question types supported

- Multiple choice
- True/false
- Multiselect
- Matching
- Short answer
- Order blocks
- Coding

Randomization is supported for all question types except for coding questions.

## Features

- For coding questions, produces all autograder tests with partial credit through test design, and outputs a sample solution for you to test it out
- Outputs the code and instructions to generate a UML class diagram, linked list diagram, or binary tree diagram if applicable to the question
- Only references Java and DSA1 course topics
- Adds the appropriate topics and tags to `info.json`
- Asks for follow up information if it is missing any details

## Usage

### Writing a new question

1. In PrairieLearn's localhost interface, create a new question.
2. Tell the agent what you want in the question, including the question type, topic, difficulty, whether there is randomization, and QID of the newly created question.

### Editing an existing question

Tell the agent what you want to edit in the question, and include the QID.

### Example prompts

```
Write a hard coding question on linked list reversal. QID is reverseLinkedList. The question should be worth 25 points.
```
 
```
Make a randomized true/false question pool on AVL tree properties. QID: avlTrueFalse.
```
 
```
Add a matching question comparing abstract classes vs interfaces. QID: abstractVsInterface. Show 6 of the 9 statements each attempt.
```

```
Modify the question with QID arrListRemoveDupes to ask the student to sort the list in descending order instead of ascending.
```

## Installation

This skill has already been added to the `pl-virginia-cs2100` project folder.

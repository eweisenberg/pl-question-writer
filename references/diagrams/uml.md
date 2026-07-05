# UML Class Diagram

## Instructions
1. Go to https://editor.plantuml.com
2. Paste the generated PlantUML code below into the left pane; right pane previews live.
3. Export PNG using the 📷 icon.
4. Place the file in `questions/<QID>/clientFilesQuestion/<filename>.png`
5. Embed in question.html: `<pl-figure file-name="<filename>.png" inline="true"></pl-figure>`

## Template
```
@startuml
hide circle
skinparam classAttributeIconSize 0

class ClassName {
    +publicField: Type
    -privateField: Type
    +ClassName(param: Type)
    +method(): ReturnType
}

' Inheritance: B extends A
' A <|-- B

' Interface implementation: A implements B
' B <|.. A
@enduml
```

## Notes
- `+` = public, `-` = private, `#` = protected
- `A <|-- B` means B extends A (inheritance)
- `B <|.. A` means A implements B (interface)
- When generating a diagram for a question, produce a filled-in PlantUML snippet (not the template) matching the exact class(es) in the question

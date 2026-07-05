# Coding Question Reference

Coding questions use Java + JUnit 5 external grading. The student writes Java code in a
browser editor, which is compiled and tested against hidden JUnit tests.

---

## File Structure

```
questionName/
├── info.json
├── question.html
├── server.py
└── tests/
    ├── junit/
    │   └── Tests.java
    └── libs/          ← extra/provided Java classes go here (do NOT modify server.py)
        └── MyClass.java
```

Extra Java files (helper classes, provided superclasses, etc.) go in `tests/libs/` as `.java`
files. Do **not** add anything to `server.py` for these — the grader picks them up automatically.
If the question provides images, they go in `clientFilesQuestion/`.

---

## server.py (Standard)

For a single-file solution (most common):
```python
def generate(data):
    data["params"]["_grader"] = {
        "type": "java",
        "workingDirectory": "tests",
        "compile": {
            "command": "javac",
            "args": ["-cp", ".:junit/*", "../student_code/Solution.java", "junit/Tests.java"]
        },
        "execute": {
            "command": "java",
            "args": ["-cp", ".:junit/*", "org.junit.runner.JUnitCore", "Tests",]
        }
    }
```

For a custom class name (e.g. `Truck.java`), replace `Solution.java` with `Truck.java` in the args. For multiple student files, list each `../student_code/FileName.java` in args before `junit/Tests.java`.

### Unallowed Methods Check

When the question forbids certain methods (e.g., `.sort()`, `.contains()`):
```python
from base64 import b64decode

def generate(data):
    data["params"]["_grader"] = { ... }  # same as above

def parse(data):
    files = {f['name']: b64decode(f['contents']).decode()
             for f in data['submitted_answers']['_files']}
    for unallowed in [".sort(", ".contains("]:
        if unallowed in files["Solution.java"]:
            data["format_errors"]["unallowed"] = unallowed
```

And in `question.html` submission panel:
```html
<pl-submission-panel>
    {{#format_errors.unallowed}}
    <h4 class="fw-bold">Your solution uses an unallowed method: <code>{{format_errors.unallowed}}</code></h4>
    <h4 class="fw-bold">Please resubmit after removing all occurrences of <code>{{format_errors.unallowed}}</code> from your code and comments.</h4>
    {{/format_errors.unallowed}}
    <pl-external-grader-results></pl-external-grader-results>
    <pl-file-preview></pl-file-preview>
</pl-submission-panel>
```

---

## question.html Structure

### Single-file solution (simple method)
```html
<pl-question-panel>
<markdown>
SCENARIO SETUP. Your task is to write a method `methodName` that DESCRIPTION.
</markdown>

<p style="font-weight: bold;">Example 1:</p>
<div style="margin-left: 20px;">
    <markdown>Input: `param = value`</markdown>
    <markdown>Output: `returnValue`</markdown>
    <markdown>Explanation: Why this input produces that output.</markdown>
</div>

<p style="font-weight: bold;">Example 2:</p>
<div style="margin-left: 20px;">
    <markdown>Input: `param = value2`</markdown>
    <markdown>Output: `returnValue2`</markdown>
    <markdown>Explanation: Edge case explanation.</markdown>
</div>

<p style="font-weight: bold;">You may assume:</p>
<div style="margin-left: 20px;">
    <markdown>List assumptions here (non-null inputs, size constraints, etc.)</markdown>
</div>

<p style="font-weight: bold;">Restrictions:</p>
<div style="margin-left: 20px;">
    <markdown>You may not use `SomeForbiddenMethod`. Delete this section if none.</markdown>
</div>

<java-quick-reference></java-quick-reference>

<pl-file-editor file-name="Solution.java" ace-mode="ace/mode/java">public class Solution {
    public returnType methodName(ParamType param) {
        // Type code here. Do not change the class or method headers.
    }
}</pl-file-editor>
<p style="margin-top: 1rem;">You may use the following main method to test your code, but it will not be graded. Anything printed out will show up in the "Printed output" section of the test results after you click Save &amp; Grade.</p>
<pl-file-editor file-name="Main.java" ace-mode="ace/mode/java">public class Main {
    public static void main(String[] args) {
        Solution sol = new Solution();
        // Example 1 from above
        // <call sol.methodName with example 1 inputs>
        System.out.println(<result>); // expected: <expectedOutput>
        // Example 2 from above (if applicable)
        // <call sol.methodName with example 2 inputs>
        System.out.println(<result>); // expected: <expectedOutput>
    }
}</pl-file-editor>
</pl-question-panel>

<pl-submission-panel>
    <pl-external-grader-results></pl-external-grader-results>
    <pl-file-preview></pl-file-preview>
</pl-submission-panel>
```

### Recursive method (with helper signature in starter code)
When the solution requires recursion, expose the helper signature:
```java
public class Solution {
    public int findMax(int[] arr) {
        return findMax(arr, 0);
    }
    protected int findMax(int[] arr, int index) {
        // Type code here. Do not change the class or method headers.
    }
}
```
Add **"Your solution must be recursive."** to the question prompt.

### Class implementation question
When students implement an entire class (not just a method):
```html
<pl-file-editor file-name="MyClass.java" ace-mode="ace/mode/java">public class MyClass extends ParentClass {
    // instance variables here

    // Default constructor
    public MyClass() {
        // Type code here.
    }

    // Other methods
    @Override
    public String toString() {
        // Type code here.
    }
}</pl-file-editor>
```

### Code shown with pl-code (read only)
When showing reference code that the student should NOT edit:
```html
<pl-code language="java">
public class ListNode {
    int data;
    ListNode next;
}
</pl-code>
```
Note: Use `&lt;` for `<` and `&gt;` for `>` inside pl-code blocks, including generics (e.g. `ArrayList&lt;String&gt;`).

---

## Tests.java Patterns

### Imports
Always include at the top of Tests.java:
```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import static org.junit.jupiter.api.Assertions.*;
import java.io.*;
```
Add as needed:
- `import java.util.*;` — if the question or tester uses ArrayList, HashMap, etc.
- `import java.lang.reflect.*;` — for class-implementation questions (see reflection section)

### Basic test structure
```java
public class Tests {
    @Test
    @DisplayName("Example 1")
    @Tag("points=2")  // can also be decimal e.g. points=2.5
    void example1Test() {
        Solution sol = new Solution();
        assertEquals(expectedValue, sol.methodName(input));
    }
}
```

### Point allocation rules

**Always ask the user for the total point value before writing Tests.java.**
If not provided, ask: "How many total points should this question be worth?"

Once you have the total, allocate as follows — values may be integers or decimals (e.g. `2.5`, `1.75`) if needed to divide points evenly. All non-zero test points must sum exactly to the specified total (the `points=0` printed output test does not count):

| Category | Allocation |
|---|---|
| Printed output test | Always `points=0` (excluded from total) |
| Example test(s) | Exactly **20% of total**, split evenly. Round to nearest whole number; adjust the last example by ±1 if needed to hit exactly 20%. |
| Typical test cases | **Majority** of remaining points (~60–70%). Cover common, expected input shapes. |
| Edge case tests | **Minority** of remaining points (~30–40%). Cover boundary conditions, empty/single inputs, extremes. |

**Worked example — 20 pts, 2 examples:**
- Printed output: 0 · Example 1: 2 · Example 2: 2 (= 20% of 20)
- Remaining 16 pts: ~10–11 to typical cases, ~5–6 to edge cases
**Partial credit through test design:** Split behavior into multiple small independent tests ordered simplest → most complex. Never make a single test worth the majority of points. For recursive questions, weight the recursion-check test like a typical test case.

### Printed output test (include when question has Main.java)
```java
// Inside Tests class (java.io.* already imported at top):
@Test
@DisplayName("Printed output")
@Tag("points=0")
void print() {
    PrintStream originalOut = System.out;
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    System.setOut(new PrintStream(baos));
    System.out.println("\n===========================");
    Main.main(new String[0]);
    System.setOut(originalOut);
    throw new Printed_Output_Shown_Below(baos.toString());
}
```
Note: `Printed_Output_Shown_Below` is a custom exception class already in the test environment.

### Recursion check (CallCounter pattern)
When question requires a recursive solution, include a CallCounter subclass:
```java
@Test
@DisplayName("Solution is recursive")
@Tag("points=5")
void isRecursiveTest() {
    int arr[] = {89, 13, 74, 128};
    CallCounter cc = new CallCounter();
    int ignored = cc.houseWithMostCandy(arr);
    assertTrue(cc.numCalls > 1);
}
```
And define CallCounter inside Tests.java:
```java
class CallCounter extends Solution {
    int numCalls = 0;
    @Override
    protected int houseWithMostCandy(int[] houses, int index) {
        numCalls++;
        return super.houseWithMostCandy(houses, index);
    }
}
```
Note: The protected helper method in Solution must match the overridden method here.

### Common assertion patterns
Use standard JUnit 5 assertions: `assertEquals`, `assertNotEquals`, `assertTrue`, `assertFalse`, `assertNull`, `assertNotNull`, `assertArrayEquals`, `assertThrows`. See examples below for usage.

### Multi-file test (when student file depends on a provided class)
Place the provided class in `tests/libs/` — the file structure section above has the full details.

---

### Class implementation with reflection (student writes multiple methods)

When the student must implement a class with methods not given in the starter code, use
`import java.lang.reflect.*;` so a missing method fails only its own test without preventing
the rest of `Tests.java` from compiling. See **Example 2** below for the full pattern.

Key rules:
- Wrap constructor lookup in `try/catch NoSuchMethodException`; call `fail(...)` then `return`
- Use primitive class literals for params: `int.class`, `double.class`, `boolean.class`
- Put each method under test in its own `@Test` for partial credit; only combine when tightly coupled
- Direct calls are fine once the object is constructed — use reflection only for methods the student may not have written yet

---

## Common Mistakes to Avoid

- Don't use `@BeforeEach` unless truly needed (each test should be self-contained)
- Leave all method bodies blank (`// Type code here. Do not change the class or method headers.`) — do not add placeholder return statements. The student's task is to write code that compiles.
- Always match the method signature in Tests.java exactly to the starter code in question.html
- `singleVariant: true` must be in info.json for coding questions (prevents re-randomization)

---

## Examples

### Example 1: Recursive Method (trickOrTreat)

**Demonstrates:** Recursive method with public/protected helper, CallCounter pattern, creative scenario.

**question.html:**
```html
<pl-question-panel>
    <markdown>Trick or treat! This Halloween, you want to scope out the house that is giving out
    the most candy. You have an array of ints representing the number of pieces of candy that
    the owner of each house bought to give out this year. Given this integer array, write a method
    `houseWithMostCandy` that returns the index of the house that is giving out the most candy.
    **Your solution must be recursive.**
    </markdown>

    <p style="font-weight: bold;">Example 1:</p>
    <div style="margin-left: 20px;">
        <markdown>Input: `houses = {89, 13, 74, 128}`</markdown>
        <markdown>Output: `3`</markdown>
        <markdown>Explanation: The house at index 3 is giving out the most candy with 128 pieces.</markdown>
    </div>

    <p style="font-weight: bold;">You may assume:</p>
    <div style="margin-left: 20px;">
        <markdown>All values in `houses` will be non-negative. `houses` will contain at least
        one element. `houses` will not contain any ties for most candy.</markdown>
    </div>

    <java-quick-reference></java-quick-reference>

    <pl-file-editor file-name="Solution.java" ace-mode="ace/mode/java">public class Solution {
    public int houseWithMostCandy(int[] houses){
        return houseWithMostCandy(houses, 0);
    }
    protected int houseWithMostCandy(int[] houses, int index){
        // Type code here. Do not change the class or method headers.
    }
}</pl-file-editor>
<p style="margin-top: 1rem;">You may use the following main method to test your code, but it will not be graded. Anything printed out will show up in the "Printed output" section of the test results after you click Save &amp; Grade.</p>
<pl-file-editor file-name="Main.java" ace-mode="ace/mode/java">public class Main {
    public static void main(String[] args) {
        Solution sol = new Solution();
        // Example 1 from above
        int[] houses = {89, 13, 74, 128};
        System.out.println(sol.houseWithMostCandy(houses)); // expected: 3
    }
}</pl-file-editor>
</pl-question-panel>

<pl-submission-panel>
    <pl-external-grader-results></pl-external-grader-results>
    <pl-file-preview></pl-file-preview>
</pl-submission-panel>
```

**Tests.java:**
```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import static org.junit.jupiter.api.Assertions.*;
import java.io.*;

public class Tests {
    @Test @DisplayName("Printed output") @Tag("points=0")
    void print() {
        PrintStream originalOut = System.out;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        System.setOut(new PrintStream(baos));
        System.out.println("\n===========================");
        Main.main(new String[0]);
        System.setOut(originalOut);
        throw new Printed_Output_Shown_Below(baos.toString());
    }

    @Test @DisplayName("Example 1") @Tag("points=1")
    void example1Test(){
        int arr[] = {89, 13, 74, 128};
        Solution sol = new Solution();
        assertEquals(3, sol.houseWithMostCandy(arr));
    }

    @Test @DisplayName("Solution is recursive") @Tag("points=7")
    void isRecursiveTest(){
        int arr[] = {89, 13, 74, 128};
        CallCounter cc = new CallCounter();
        int ignored = cc.houseWithMostCandy(arr);
        assertTrue(cc.numCalls > 1);
    }

    @Test @DisplayName("One house") @Tag("points=1")
    void oneHouseTest(){
        int arr[] = {73};
        Solution sol = new Solution();
        assertEquals(0, sol.houseWithMostCandy(arr));
    }
}

class CallCounter extends Solution {
    int numCalls = 0;
    @Override
    protected int houseWithMostCandy(int[] houses, int index) {
        numCalls++;
        return super.houseWithMostCandy(houses, index);
    }
}
```

---

### Example 2: Class Implementation with Provided Superclass (vehicleInh)

**Demonstrates:** Student writes a full class; provided superclass in `tests/libs/`; `toString` + `equals`; reflection pattern.

**question.html:**
```html
<pl-question-panel>
<markdown>
You are given the `Vehicle` class (shown in the UML diagram below). Implement the `Truck` class
that extends `Vehicle`. Your `Truck` class must have a constructor, a `toString` method, and an
`equals` method as described in the examples.
</markdown>

<pl-figure file-name="uml.png" inline="true"></pl-figure>

<p style="font-weight: bold;">Example 1 (toString):</p>
<div style="margin-left: 20px;">
    <markdown>Input: `new Truck("Diesel", 18)`</markdown>
    <markdown>Output of `toString()`: `"fuelType: Diesel, numWheels: 18"`</markdown>
</div>

<p style="font-weight: bold;">Example 2 (equals):</p>
<div style="margin-left: 20px;">
    <markdown>`new Truck("Electric", 4).equals(new Truck("Electric", 4))` → `true`</markdown>
    <markdown>`new Truck("Diesel", 18).equals(new Truck("Electric", 18))` → `false`</markdown>
</div>

<java-quick-reference></java-quick-reference>

<pl-file-editor file-name="Truck.java" ace-mode="ace/mode/java">public class Truck extends Vehicle {
    // instance variables here

    public Truck(String fuelType, int numWheels) {
        // Type code here.
    }

    @Override
    public String toString() {
        // Type code here.
    }

    @Override
    public boolean equals(Object o) {
        // Type code here.
    }
}</pl-file-editor>
<p style="margin-top: 1rem;">You may use the following main method to test your code, but it will not be graded. Anything printed out will show up in the "Printed output" section of the test results after you click Save &amp; Grade.</p>
<pl-file-editor file-name="Main.java" ace-mode="ace/mode/java">public class Main {
    public static void main(String[] args) {
        // Example 1 from above
        Truck t1 = new Truck("Diesel", 18);
        System.out.println(t1.toString()); // expected: fuelType: Diesel, numWheels: 18
        // Example 2 from above
        Truck t2 = new Truck("Electric", 4);
        Truck t3 = new Truck("Electric", 4);
        System.out.println(t2.equals(t3));  // expected: true
        System.out.println(t1.equals(t2));  // expected: false
    }
}</pl-file-editor>
</pl-question-panel>

<pl-submission-panel>
    <pl-external-grader-results></pl-external-grader-results>
    <pl-file-preview></pl-file-preview>
</pl-submission-panel>
```

**Image:** Generate `uml.png` using the UML diagram tool (see SKILL.md) and place in `questions/<QID>/clientFilesQuestion/uml.png`. `Vehicle.java` lives in `tests/libs/Vehicle.java`.

**Tests.java** (exact file from the course — reflects the full reflection pattern for class implementation questions):
```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import static org.junit.jupiter.api.Assertions.*;
import java.io.*;
import java.lang.reflect.*;

public class Tests {

    @Test
    @DisplayName("Example")
    @Tag("points=10")
    void example() throws Exception {
        Truck t1 = null;
        Truck t2 = null;

        try {
            // 1. Check/Invoke Constructor: Test will fail here if the 2-arg constructor is missing.
            // Note: int.class is used for the primitive integer parameter
            Constructor<Truck> constructor = Truck.class.getDeclaredConstructor(String.class, int.class);

            // Instantiate t1 and t2 using reflection
            t1 = constructor.newInstance("Diesel", 18);
            t2 = constructor.newInstance("Electric", 4);

        } catch (NoSuchMethodException e) {
            // Fail if the constructor is missing
            fail("Test Failed: Cannot run example tests because the required two-parameter constructor (String, int) is missing.");
            return;
        }

        // 2. Proceed with toString check using reflection
        Method toStringMethod = Truck.class.getMethod("toString");
        String str = (String) toStringMethod.invoke(t1);

        // 3. Proceed with equals check using reflection
        Method equalsMethod = Truck.class.getMethod("equals", Object.class);
        boolean isEqual = (Boolean) equalsMethod.invoke(t1, t2);

        assertEquals("fuelType: Diesel, numWheels: 18", str);
        assertFalse(isEqual);
    }

    @Test
    @DisplayName("numWheels field")
    @Tag("points=2.5")
    void numWheelsField() {
        Class<Truck> truck = Truck.class;
        Field[] allFields = truck.getDeclaredFields();
        assertEquals(1, allFields.length);
        Field field = allFields[0];
        assertEquals("numWheels", field.getName());
        assertEquals(int.class, field.getType());
        assertEquals(Modifier.PRIVATE, field.getModifiers());
    }

    @Test
    @DisplayName("Default constructor")
    @Tag("points=7.5")
    void defaultConstructor() throws Exception {
        // Truck t = new Truck();
        Truck t = null;
        try {
            // 1. Check/Invoke Constructor
            Constructor<Truck> constructor = Truck.class.getDeclaredConstructor();
            t = constructor.newInstance();
        } catch (NoSuchMethodException e) {
            // Fail if the constructor is missing
            fail();
            return;
        }

        Field numWheelsField = Truck.class.getDeclaredField("numWheels");
        numWheelsField.setAccessible(true);
        assertNotNull(numWheelsField.get(t));
        numWheelsField.setAccessible(false);

        Field fuelTypeField = Vehicle.class.getDeclaredField("fuelType");
        fuelTypeField.setAccessible(true);
        assertNotNull(fuelTypeField.get(t));
        fuelTypeField.setAccessible(false);
    }

    @Test
    @DisplayName("2 parameter constructor")
    @Tag("points=7.5")
    void twoParameterConstructor() throws Exception {
        Truck t = null;
        int expectedWheels = 10;
        String expectedFuel = "Hydrogen";

        try {
            // 1. Check/Invoke Constructor
            Constructor<Truck> constructor = Truck.class.getDeclaredConstructor(String.class, int.class);
            t = constructor.newInstance(expectedFuel, expectedWheels);
        } catch (NoSuchMethodException e) {
            // Fail if the constructor is missing
            fail();
            return;
        }

        // 2. Check 'numWheels' field value
        try {
            Field numWheelsField = Truck.class.getDeclaredField("numWheels");
            numWheelsField.setAccessible(true);
            assertEquals(expectedWheels, numWheelsField.get(t));
            numWheelsField.setAccessible(false);
        } catch (NoSuchFieldException e) {
            fail();
        }

        // 3. Check 'fuelType' field value (inherited from Vehicle)
        try {
            Field fuelTypeField = Vehicle.class.getDeclaredField("fuelType");
            fuelTypeField.setAccessible(true);
            assertEquals(expectedFuel, fuelTypeField.get(t));
            fuelTypeField.setAccessible(false);
        } catch (NoSuchFieldException e) {
            fail();
        }
    }

    @Test
    @DisplayName("toString")
    @Tag("points=10")
    void toStringMethod() throws Exception {
        Truck t1 = null;
        Truck t2 = null;

        try {
            // 1. Check/Invoke Constructor
            Constructor<Truck> constructor = Truck.class.getDeclaredConstructor(String.class, int.class);

            // Instantiate t1 using reflection
            t1 = constructor.newInstance("Diesel", 8);
            // Instantiate t2 using reflection
            t2 = constructor.newInstance("DIESEL", 8);

        } catch (NoSuchMethodException e) {
            fail("Must write 2 argument constructor first");
            return;
        }

        // 2. Proceed with toString check using reflection
        Method toStringMethod = Truck.class.getMethod("toString");

        // Check t1
        String result1 = (String) toStringMethod.invoke(t1);
        assertEquals("fuelType: Diesel, numWheels: 8", result1);

        // Check t2
        String result2 = (String) toStringMethod.invoke(t2);
        assertEquals("fuelType: DIESEL, numWheels: 8", result2);
    }

    @Test
    @DisplayName("equals true")
    @Tag("points=5")
    void equalsTrue() throws Exception {
        Truck a, b, c, d;
        try {
            // Check/Invoke Constructor
            Constructor<Truck> constructor = Truck.class.getDeclaredConstructor(String.class, int.class);

            a = constructor.newInstance("diesel", 6);
            b = constructor.newInstance("Diesel", 6);
            c = constructor.newInstance("diesel", 6);
            d = constructor.newInstance("DIESEL", 6);

        } catch (NoSuchMethodException e) {
            fail("Must write 2 argument constructor first");
            return;
        }

        Method equalsMethod = Truck.class.getMethod("equals", Object.class);

        // All calls use reflection's invoke()
        assertTrue((Boolean) equalsMethod.invoke(a, b));
        assertTrue((Boolean) equalsMethod.invoke(b, a));
        assertTrue((Boolean) equalsMethod.invoke(a, c));
        assertTrue((Boolean) equalsMethod.invoke(c, a));
        assertTrue((Boolean) equalsMethod.invoke(a, d));
        assertTrue((Boolean) equalsMethod.invoke(d, a));
        assertTrue((Boolean) equalsMethod.invoke(b, c));
        assertTrue((Boolean) equalsMethod.invoke(c, b));
        assertTrue((Boolean) equalsMethod.invoke(b, d));
        assertTrue((Boolean) equalsMethod.invoke(d, b));
        assertTrue((Boolean) equalsMethod.invoke(c, d));
        assertTrue((Boolean) equalsMethod.invoke(d, c));
    }

    @Test
    @DisplayName("equals false")
    @Tag("points=5")
    void equalsFalse() throws Exception {
        Truck a, b, c, d;
        try {
            // Check/Invoke Constructor
            Constructor<Truck> constructor = Truck.class.getDeclaredConstructor(String.class, int.class);

            a = constructor.newInstance("diesel", 6);
            b = constructor.newInstance("diesel", 18);
            c = constructor.newInstance("electric", 6);
            d = constructor.newInstance("electric", 18);

        } catch (NoSuchMethodException e) {
            fail("Must write 2 argument constructor first");
            return;
        }

        Method equalsMethod = Truck.class.getMethod("equals", Object.class);

        // All calls use reflection's invoke()
        assertFalse((Boolean) equalsMethod.invoke(a, b));
        assertFalse((Boolean) equalsMethod.invoke(b, a));
        assertFalse((Boolean) equalsMethod.invoke(a, c));
        assertFalse((Boolean) equalsMethod.invoke(c, a));
        assertFalse((Boolean) equalsMethod.invoke(a, d));
        assertFalse((Boolean) equalsMethod.invoke(d, a));
        assertFalse((Boolean) equalsMethod.invoke(b, c));
        assertFalse((Boolean) equalsMethod.invoke(c, b));
        assertFalse((Boolean) equalsMethod.invoke(b, d));
        assertFalse((Boolean) equalsMethod.invoke(d, b));
        assertFalse((Boolean) equalsMethod.invoke(c, d));
        assertFalse((Boolean) equalsMethod.invoke(d, c));
    }

    @Test
    @DisplayName("Proper inheritance")
    @Tag("points=2.5")
    void properInheritance() {
        assertTrue(Vehicle.class.isAssignableFrom(Truck.class));
    }

    @Test
    @DisplayName("Printed output")
    @Tag("points=0")
    void print() {
        PrintStream originalOut = System.out;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        System.setOut(new PrintStream(baos));
        System.out.println("\n===========================");
        Main.main(new String[0]);
        System.setOut(originalOut);
        throw new Printed_Output_Shown_Below(baos.toString());
    }
}
```

---

### Example 3: Single Method in Provided Class — equals (undergradEquals)

**Demonstrates:** Student implements one method inside a fully provided class skeleton; null and wrong-type edge cases.

**question.html:**
```html
<pl-question-panel>
<markdown>
Implement the `equals` method for the `Undergrad` class below. Two `Undergrad` objects are equal
if and only if they have the same `compID` and `year`.
</markdown>

<p style="font-weight: bold;">Example 1:</p>
<div style="margin-left: 20px;">
    <markdown>`new Undergrad("abc1de", 2).equals(new Undergrad("abc1de", 2))` → `true`</markdown>
</div>

<p style="font-weight: bold;">Example 2:</p>
<div style="margin-left: 20px;">
    <markdown>`new Undergrad("abc1de", 2).equals(new Undergrad("abc1de", 3))` → `false`</markdown>
</div>

<java-quick-reference></java-quick-reference>

<pl-file-editor file-name="Undergrad.java" ace-mode="ace/mode/java">import java.util.Objects;

public class Undergrad {
    private String compID = "";
    private int year = 1;

    public Undergrad(String compID, int year) { this.compID = compID; this.year = year; }
    public String getCompID() { return compID; }
    public int getYear() { return year; }

    @Override
    public boolean equals(Object o) {
        // Type code here. Do not change the method header or code outside this method.
    }

    // You may ignore the following method
    @Override public int hashCode() { return Objects.hash(compID, year); }
}</pl-file-editor>
<p style="margin-top: 1rem;">You may use the following main method to test your code, but it will not be graded. Anything printed out will show up in the "Printed output" section of the test results after you click Save &amp; Grade.</p>
<pl-file-editor file-name="Main.java" ace-mode="ace/mode/java">public class Main {
    public static void main(String[] args) {
        // Example 1 from above
        Undergrad u1 = new Undergrad("abc1de", 2);
        Undergrad u2 = new Undergrad("abc1de", 2);
        System.out.println(u1.equals(u2)); // expected: true
        // Example 2 from above
        Undergrad u3 = new Undergrad("abc1de", 3);
        System.out.println(u1.equals(u3)); // expected: false
    }
}</pl-file-editor>
</pl-question-panel>

<pl-submission-panel>
    <pl-external-grader-results></pl-external-grader-results>
    <pl-file-preview></pl-file-preview>
</pl-submission-panel>
```

**Tests.java — always include null and wrong-type edge cases for equals questions:**
```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Tag;
import static org.junit.jupiter.api.Assertions.*;
import java.io.*;

public class Tests {
    @Test @DisplayName("Printed output") @Tag("points=0")
    void print() {
        PrintStream originalOut = System.out;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        System.setOut(new PrintStream(baos));
        System.out.println("\n===========================");
        Main.main(new String[0]);
        System.setOut(originalOut);
        throw new Printed_Output_Shown_Below(baos.toString());
    }

    @Test @DisplayName("Example 1 — equal") @Tag("points=2")
    void example1Test() {
        Undergrad u1 = new Undergrad("abc1de", 2);
        Undergrad u2 = new Undergrad("abc1de", 2);
        assertEquals(u1, u2);
    }

    @Test @DisplayName("Example 2 — not equal (different year)") @Tag("points=2")
    void example2Test() {
        Undergrad u1 = new Undergrad("abc1de", 2);
        Undergrad u3 = new Undergrad("abc1de", 3);
        assertNotEquals(u1, u3);
    }

    @Test @DisplayName("Null") @Tag("points=1")
    void nullTest() {
        Undergrad ug = new Undergrad("def4gh", 3);
        assertNotEquals(null, ug);
    }

    @Test @DisplayName("Wrong type") @Tag("points=2")
    void otherType() {
        Undergrad ug = new Undergrad("jkl5mn", 1);
        assertNotEquals("jkl5mn", ug);
    }
}
```

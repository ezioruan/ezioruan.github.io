---
title: "Exploring Structural Pattern Matching in Python 3.10"
date: 2022-02-18 09:00:00
tags: Python, Python3.10, Pattern Matching
---

Python 3.10 introduced a host of new features, but perhaps the most talked-about is Structural Pattern Matching. This feature, introduced in PEP 634, adds a `match` statement that behaves like a `switch` statement found in other languages, but with much more power and flexibility. It allows you to match variables against a variety of patterns, making complex conditional logic more readable and less error-prone.

<!--more-->

### Beyond `if/elif/else`

For years, Python developers have relied on long `if/elif/else` chains to handle branching logic based on the value or type of a variable. While functional, this can become clumsy.

Consider a function that processes a command:

```python
def process_command(command):
    if command == "start":
        print("Starting...")
    elif command == "stop":
        print("Stopping...")
    elif isinstance(command, tuple) and command[0] == "move":
        _, x, y = command
        print(f"Moving to ({x}, {y})")
    else:
        print("Unknown command")
```

This code works, but it's a mix of value checks, type checks, and unpacking. Structural Pattern Matching provides a cleaner way to express this.

### The `match` Statement

Let's rewrite the above example using `match`:

```python
def process_command_match(command):
    match command:
        case "start":
            print("Starting...")
        case "stop":
            print("Stopping...")
        case ("move", x, y):
            print(f"Moving to ({x}, {y})")
        case _:
            print("Unknown command")
```

This is immediately more readable. The `match` statement takes a subject (`command`), and each `case` block defines a pattern to match against it. The first pattern that matches is executed. The underscore (`_`) is a wildcard pattern that matches anything, serving as a default case.

### Key Patterns

Structural Pattern Matching is powerful because it can match on the *structure* of the data.

#### 1. Literal and Capture Patterns

As seen above, you can match against literal values like `"start"`. You can also capture parts of a pattern into variables, like `x` and `y` in the `("move", x, y)` pattern.

#### 2. Sequence Patterns

You can match sequences like lists or tuples. You can use `*` to capture multiple items.

```python
match items:
    case [item1, item2]:
        print(f"Found two items: {item1}, {item2}")
    case [item1, *rest]:
        print(f"Found {item1} and {len(rest)} other items.")
    case []:
        print("Found an empty list.")
```

#### 3. Mapping Patterns

You can match dictionaries (or other mappings) based on their keys and values.

```python
match data:
    case {"name": str(name), "age": int(age)}:
        print(f"{name} is {age} years old.")
    case {"error": message}:
        print(f"Error: {message}")
```

This pattern not only checks for the presence of the "name" and "age" keys but also captures their values into variables and checks their types.

#### 4. Class Patterns

Perhaps the most powerful feature is matching against class instances and their attributes.

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

def locate(p):
    match p:
        case Point(x=0, y=0):
            print("The point is at the origin.")
        case Point(x=x, y=0):
            print(f"The point is on the x-axis at {x}.")
        case Point(x=0, y=y):
            print(f"The point is on the y-axis at {y}.")
        case Point(x, y):
            print(f"The point is at ({x}, {y}).")
        case _:
            print("Not a point.")

locate(Point(5, 0))  # Output: The point is on the x-axis at 5.
```

### Conclusion

Structural Pattern Matching is a significant addition to the Python language. It provides a structured, readable, and powerful way to handle complex conditional logic that goes far beyond a simple `switch` statement. By allowing you to match on the shape and value of your data, it helps you write code that is more expressive and easier to maintain. If you are using Python 3.10 or newer, it's a feature well worth exploring for your next project.

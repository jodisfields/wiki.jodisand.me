# Python

Python is a high-level, interpreted programming language known for its simplicity and readability.

## Installation

```bash
# Check version
python3 --version
pip3 --version

# Using virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

## Basic Syntax

```python
# Variables
name = "John"
age = 30
height = 5.9
is_active = True

# Print
print(f"Hello, {name}!")

# Comments
# This is a single-line comment

"""
This is a
multi-line comment
"""
```

## Data Types

```python
# String
text = "Hello World"
multi_line = """Multiple
lines"""

# Numbers
integer = 42
floating = 3.14
complex_num = 1 + 2j

# Boolean
is_true = True
is_false = False

# List
fruits = ["apple", "banana", "cherry"]

# Tuple
coordinates = (10, 20)

# Dictionary
person = {"name": "John", "age": 30}

# Set
unique_numbers = {1, 2, 3}

# None
value = None
```

## Operators

```python
# Arithmetic
result = 10 + 5  # 15
result = 10 - 5  # 5
result = 10 * 5  # 50
result = 10 / 5  # 2.0
result = 10 // 3  # 3 (floor division)
result = 10 % 3  # 1 (modulo)
result = 2 ** 3  # 8 (exponent)

# Comparison
is_equal = (5 == 5)  # True
is_not_equal = (5 != 3)  # True
is_greater = (5 > 3)  # True
is_less = (3 < 5)  # True

# Logical
and_result = True and False  # False
or_result = True or False  # True
not_result = not True  # False

# Identity
is_same = (a is b)
is_not_same = (a is not b)

# Membership
is_in = ("a" in ["a", "b"])  # True
is_not_in = ("c" not in ["a", "b"])  # True
```

## Control Flow

```python
# If statement
if age >= 18:
    print("Adult")
elif age >= 13:
    print("Teenager")
else:
    print("Child")

# For loop
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

for fruit in fruits:
    print(fruit)

# While loop
count = 0
while count < 5:
    print(count)
    count += 1

# Break and continue
for i in range(10):
    if i == 5:
        break
    if i % 2 == 0:
        continue
    print(i)
```

## Functions

```python
# Basic function
def greet(name):
    return f"Hello, {name}!"

# Default parameters
def power(base, exponent=2):
    return base ** exponent

# Variable arguments
def sum_all(*args):
    return sum(args)

# Keyword arguments
def display_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

# Lambda functions
square = lambda x: x ** 2
add = lambda a, b: a + b
```

## Lists

```python
# Create list
numbers = [1, 2, 3, 4, 5]

# Access elements
first = numbers[0]
last = numbers[-1]

# Slicing
subset = numbers[1:4]  # [2, 3, 4]
reversed_list = numbers[::-1]

# Methods
numbers.append(6)
numbers.insert(0, 0)
numbers.remove(3)
popped = numbers.pop()
numbers.sort()
numbers.reverse()

# List comprehension
squares = [x ** 2 for x in range(10)]
evens = [x for x in range(10) if x % 2 == 0]
```

## Dictionaries

```python
# Create dictionary
person = {
    "name": "John",
    "age": 30,
    "city": "New York"
}

# Access
name = person["name"]
age = person.get("age")

# Modify
person["age"] = 31
person["email"] = "john@example.com"

# Delete
del person["city"]
removed = person.pop("email")

# Iterate
for key in person:
    print(f"{key}: {person[key]}")

for key, value in person.items():
    print(f"{key}: {value}")

# Dictionary comprehension
squares = {x: x ** 2 for x in range(5)}
```

## String Methods

```python
text = "Hello World"

# Case
upper = text.upper()  # "HELLO WORLD"
lower = text.lower()  # "hello world"
title = text.title()  # "Hello World"

# Search
starts = text.startswith("Hello")  # True
ends = text.endswith("World")  # True
index = text.find("World")  # 6
count = text.count("l")  # 3

# Modify
replaced = text.replace("World", "Python")
stripped = "  text  ".strip()
split = text.split()  # ["Hello", "World"]
joined = "-".join(["a", "b", "c"])  # "a-b-c"

# Format
formatted = f"Name: {name}, Age: {age}"
formatted = "Name: {}, Age: {}".format(name, age)
```

## File I/O

```python
# Write file
with open("file.txt", "w") as f:
    f.write("Hello World\n")
    f.write("Second line\n")

# Read file
with open("file.txt", "r") as f:
    content = f.read()

# Read lines
with open("file.txt", "r") as f:
    lines = f.readlines()

# Append
with open("file.txt", "a") as f:
    f.write("Appended line\n")

# JSON
import json

# Write JSON
data = {"name": "John", "age": 30}
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# Read JSON
with open("data.json", "r") as f:
    data = json.load(f)
```

## Classes

```python
class Person:
    # Class variable
    species = "Homo sapiens"

    # Constructor
    def __init__(self, name, age):
        self.name = name
        self.age = age

    # Instance method
    def greet(self):
        return f"Hello, I'm {self.name}"

    # Class method
    @classmethod
    def from_birth_year(cls, name, birth_year):
        return cls(name, 2024 - birth_year)

    # Static method
    @staticmethod
    def is_adult(age):
        return age >= 18

# Create instance
person = Person("John", 30)
print(person.greet())

# Inheritance
class Student(Person):
    def __init__(self, name, age, student_id):
        super().__init__(name, age)
        self.student_id = student_id

    def study(self):
        return f"{self.name} is studying"
```

## Exception Handling

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
except Exception as e:
    print(f"Error: {e}")
else:
    print("No exceptions")
finally:
    print("Always executed")

# Raise exception
def validate_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    return age
```

## Modules

```python
# Import entire module
import math
print(math.sqrt(16))

# Import specific items
from math import sqrt, pi
print(sqrt(16))

# Import with alias
import numpy as np
array = np.array([1, 2, 3])

# Create module (mymodule.py)
def add(a, b):
    return a + b

# Use module
import mymodule
result = mymodule.add(5, 3)
```

## List Comprehensions

```python
# Basic
squares = [x ** 2 for x in range(10)]

# With condition
evens = [x for x in range(10) if x % 2 == 0]

# Multiple conditions
filtered = [x for x in range(20) if x % 2 == 0 if x % 3 == 0]

# Nested loops
pairs = [(x, y) for x in range(3) for y in range(3)]

# Dictionary comprehension
squares_dict = {x: x ** 2 for x in range(5)}

# Set comprehension
unique_squares = {x ** 2 for x in [1, -1, 2, -2]}
```

## Decorators

```python
def uppercase_decorator(func):
    def wrapper():
        result = func()
        return result.upper()
    return wrapper

@uppercase_decorator
def greet():
    return "hello world"

print(greet())  # "HELLO WORLD"

# Decorator with arguments
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def say_hello():
    print("Hello!")
```

## Common Libraries

```python
# datetime
from datetime import datetime, timedelta

now = datetime.now()
tomorrow = now + timedelta(days=1)
formatted = now.strftime("%Y-%m-%d %H:%M:%S")

# os
import os

current_dir = os.getcwd()
os.makedirs("new_folder", exist_ok=True)
files = os.listdir(".")

# random
import random

random_int = random.randint(1, 10)
random_choice = random.choice([1, 2, 3, 4, 5])
random.shuffle(my_list)

# requests
import requests

response = requests.get("https://api.example.com/data")
data = response.json()
```

## Virtual Environment

```bash
# Create virtual environment
python3 -m venv venv

# Activate
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# Install packages
pip install requests
pip install -r requirements.txt

# Freeze dependencies
pip freeze > requirements.txt

# Deactivate
deactivate
```

## Resources

- [Official Documentation](https://docs.python.org/)
- [Python Package Index (PyPI)](https://pypi.org/)
- [Real Python Tutorials](https://realpython.com/)

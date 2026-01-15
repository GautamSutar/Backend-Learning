# 🐍 Python Programming Fundamentals - Complete Guide

> **The most important foundation for becoming a real Python developer**

This comprehensive guide covers everything you need to master Python fundamentals, with clear theory, practical examples, real-world applications, and a complete mini-project.

---

## 📚 Table of Contents

1. [Python Syntax and Semantics](#1️⃣-python-syntax-and-semantics)
2. [Variables and Data Types](#2️⃣-variables-and-data-types)
3. [Control Flow Statements](#3️⃣-control-flow-statements)
4. [Functions and Lambda Expressions](#4️⃣-functions-and-lambda-expressions)
5. [Modules and Packages](#5️⃣-modules-and-packages)
6. [Exception Handling](#6️⃣-exception-handling)
7. [File Handling](#7️⃣-file-handling)
8. [Virtual Environments](#8️⃣-virtual-environments)
9. [Python Standard Library](#9️⃣-python-standard-library)
10. [Code Formatting and Linting](#🔟-code-formatting-and-linting)
11. [Mini Project - Student Record Management](#🚀-mini-project)

---

## 1️⃣ Python Syntax and Semantics

**Level:** Beginner

### 🔹 What is Syntax?

Syntax is **how Python code is written** (rules).

### 🔹 What is Semantics?

Semantics is **what the code means / does**.

### ✅ Key Rules

- Python uses **indentation**, not `{}`
- Code runs **line by line**
- Case-sensitive (`Age` ≠ `age`)

### ✅ Example

```python
# Syntax example
if 10 > 5:
    print("10 is greater than 5")  # correct indentation
```

❌ **Wrong:**

```python
if 10 > 5:
print("Error")  # IndentationError
```

### 🌍 Real-World Use

- Backend APIs
- Automation scripts
- AI/ML pipelines
- Web servers (Django, Flask)

---

## 2️⃣ Variables and Data Types

**Level:** Beginner

### 🔹 Variables

Used to **store data in memory**.

```python
name = "Gauti"
age = 21
is_student = True
```

### 🔹 Common Data Types

| Type  | Example    | Description |
|-------|------------|-------------|
| `int`   | `10`       | Integer numbers |
| `float` | `99.5`     | Decimal numbers |
| `str`   | `"Python"` | Text strings |
| `bool`  | `True`     | Boolean values |
| `list`  | `[1,2,3]`  | Ordered collection |
| `tuple` | `(1,2)`    | Immutable collection |
| `dict`  | `{"id":1}` | Key-value pairs |
| `set`   | `{1,2}`    | Unique values |

### ✅ Example

```python
price = 100.50
quantity = 3
total = price * quantity
print(total)
```

### 🌍 Real-World Use

- Store user data
- Store API responses
- Store database values

---

## 3️⃣ Control Flow Statements

**Level:** Beginner

Control how the program **decides and repeats**.

### 🔹 if / elif / else

```python
marks = 75

if marks >= 90:
    print("A Grade")
elif marks >= 60:
    print("B Grade")
else:
    print("Fail")
```

### 🔹 Loops

#### for loop

```python
for i in range(1, 6):
    print(i)
```

#### while loop

```python
count = 1
while count <= 5:
    print(count)
    count += 1
```

### 🌍 Real-World Use

- Checking user permissions
- Looping through database records
- Processing files

---

## 4️⃣ Functions and Lambda Expressions

**Level:** Beginner

### 🔹 Functions

Reusable blocks of code.

```python
def add(a, b):
    return a + b

print(add(10, 20))
```

### 🔹 Lambda (Short anonymous function)

```python
square = lambda x: x * x
print(square(5))
```

### 🌍 Real-World Use

- Business logic
- API handlers
- Validation functions

---

## 5️⃣ Modules and Packages

**Level:** Beginner

### 🔹 Module

A Python file (`.py`).

**math_utils.py**
```python
def multiply(a, b):
    return a * b
```

**Using the module:**
```python
import math_utils
print(math_utils.multiply(3, 4))
```

### 🔹 Package

Folder with `__init__.py`

```
utils/
 ├── __init__.py
 └── helper.py
```

### 🌍 Real-World Use

- Organizing large projects
- Clean architecture
- Reusable utilities

---

## 6️⃣ Exception Handling

**Level:** Intermediate

Used to **handle runtime errors safely**.

### ✅ Example

```python
try:
    x = int(input("Enter number: "))
    print(10 / x)
except ZeroDivisionError:
    print("Cannot divide by zero")
except ValueError:
    print("Invalid input")
finally:
    print("Execution finished")
```

### 🌍 Real-World Use

- API failure handling
- Database errors
- User input validation

---

## 7️⃣ File Handling

**Level:** Intermediate

Used to **read/write files**.

### 🔹 Write File

```python
with open("data.txt", "w") as f:
    f.write("Hello Python")
```

### 🔹 Read File

```python
with open("data.txt", "r") as f:
    print(f.read())
```

### 🌍 Real-World Use

- Logs
- Reports
- Configuration files

---

## 8️⃣ Virtual Environments

**Level:** Intermediate

Used to **isolate dependencies**.

### 🔹 Create venv

```bash
python -m venv venv
```

### 🔹 Activate

```bash
# Windows
venv\Scripts\activate

# Linux/Mac
source venv/bin/activate
```

### 🌍 Real-World Use

- Prevent version conflicts
- Professional project setup
- Required in companies

---

## 9️⃣ Python Standard Library

**Level:** Intermediate

Built-in powerful modules.

### 🔹 Examples

```python
import datetime
print(datetime.datetime.now())
```

```python
import os
print(os.getcwd())
```

```python
import math
print(math.sqrt(16))
```

### 🌍 Real-World Use

- Date handling
- File system operations
- Math & statistics

---

## 🔟 Code Formatting and Linting

**Level:** Intermediate

### 🔹 Formatting

- PEP8 style
- Clean indentation
- Proper naming

### 🔹 Tools

- `black` – auto formatter
- `flake8` – linting
- `pylint` – quality check

```bash
pip install black
black app.py
```

### 🌍 Real-World Use

- Clean readable code
- Team collaboration
- Company coding standards

---

## 🚀 MINI PROJECT

### 📌 Project: **Student Record Management System (Console App)**

A complete console application using **ALL topics (1–10)**.

### 🔹 Features

- Add student
- View students
- Save data to file
- Handle errors
- Modular structure

---

### 📂 Project Structure

```
student_project/
 ├── venv/
 ├── main.py
 ├── student.py
 ├── file_manager.py
 └── data.txt
```

---

### 📄 Code Files

#### **student.py**

```python
def create_student(name, age, marks):
    return {
        "name": name,
        "age": age,
        "marks": marks
    }
```

---

#### **file_manager.py**

```python
def save_student(student):
    with open("data.txt", "a") as f:
        f.write(str(student) + "\n")

def read_students():
    with open("data.txt", "r") as f:
        return f.readlines()
```

---

#### **main.py**

```python
from student import create_student
from file_manager import save_student, read_students

def main():
    while True:
        print("\n1. Add Student")
        print("2. View Students")
        print("3. Exit")

        try:
            choice = int(input("Choose: "))

            if choice == 1:
                name = input("Name: ")
                age = int(input("Age: "))
                marks = int(input("Marks: "))

                student = create_student(name, age, marks)
                save_student(student)
                print("Student added successfully")

            elif choice == 2:
                students = read_students()
                for s in students:
                    print(s)

            elif choice == 3:
                break
            else:
                print("Invalid choice")

        except Exception as e:
            print("Error:", e)

if __name__ == "__main__":
    main()
```

---

### 🧠 What You Learned From This Project

✔ Variables & data types  
✔ Control flow  
✔ Functions  
✔ Modules  
✔ File handling  
✔ Exception handling  
✔ Standard library  
✔ Code structure  
✔ Real-world development mindset  

---

## 🔥 Next Steps

Ready to level up? Here are your options:

1️⃣ Convert this into **Flask / Django project**  
2️⃣ Add **database (MySQL / SQLite)**  
3️⃣ Turn this into **REST API**  
4️⃣ Get **interview questions + assignments**  

---

## 💡 Tips for Success

- **Practice daily** - Consistency is key
- **Build projects** - Theory + Practice = Mastery
- **Read documentation** - Python docs are excellent
- **Join communities** - Stack Overflow, Reddit, Discord
- **Contribute to open source** - Real-world experience

---

## 📖 Additional Resources

- [Python Official Documentation](https://docs.python.org/)
- [PEP 8 Style Guide](https://pep8.org/)
- [Real Python Tutorials](https://realpython.com/)
- [Python Package Index (PyPI)](https://pypi.org/)

---

## 👨‍💻 Author

**Gauti**

---

## 📄 License

This guide is provided for educational purposes. Feel free to use, modify, and share!

---

<div align="center">

**⭐ If this helped you, give it a star! ⭐**

*Happy Coding! 🐍*

</div>
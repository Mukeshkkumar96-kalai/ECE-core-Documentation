# Phase 3: Functions

## 1. Today's Learning

- Defining and calling functions, parameters vs. arguments, `return` behavior
- Default arguments, keyword arguments, `*args` and `**kwargs`
- Argument ordering rules in function definitions
- Lambda functions and their limitations
- Variable scope (LEGB rule)
- Recursion: base case, recursive case, recursion limits

## 2. Topic Explanation

### Defining & Calling Functions
```python
def greet(name):
    """Returns a greeting string."""
    return f"Hello, {name}!"

result = greet("Zoro")
print(result)  # Hello, Zoro!
```
A function with no explicit `return` doesn't return "nothing" — it returns `None`.

### Default, Keyword & Variable-Length Arguments
```python
def power(base, exponent=2):
    return base ** exponent

power(3)                    # 9
power(3, 3)                 # 27
power(exponent=3, base=3)   # 27
```

```python
def total(*args, **kwargs):
    print(args)     # tuple of positional args
    print(kwargs)   # dict of keyword args

total(1, 2, 3, x=10, y=20)
# (1, 2, 3)
# {'x': 10, 'y': 20}
```

Argument ordering in a definition must follow:
`positional → default → *args → keyword-only → **kwargs`
Violating this order is a `SyntaxError`.

### Lambda Functions
```python
square = lambda x: x ** 2
print(square(5))  # 25

names = ["bob", "Alice", "charlie"]
print(sorted(names, key=lambda n: n.lower()))
```
A lambda body must be a single **expression** — no statements, no multi-line blocks (a conditional *expression* `a if cond else b` is allowed).

### Variable Scope (LEGB)
```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)   # local
    inner()
    print(x)       # enclosing

outer()
print(x)            # global
```

### Recursion
```python
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

print(factorial(5))  # 120
```
Every recursive function needs a base case (stops recursion) and a recursive call that moves toward it. Python's default recursion limit is 1000; exceeding it raises `RecursionError`.

## 3. Misconceptions

- **Mutable default arguments**: Assumed a fresh empty list `[]` is created on every call when a mutable default (e.g. `basket=[]`) is used. In reality, default argument values are evaluated **once**, at function-definition time, and the same object is reused/shared across all calls that don't supply their own argument.
- **`append` explains the bug**: Initially attributed changed output purely to "append adds elements" without identifying that the underlying list object was being shared across calls.
- **Scope errors look like `NameError`**: Assumed reading a global variable inside a function that also reassigns it later would work normally. In reality, Python decides variable scope at compile time by scanning the whole function body — any name assigned anywhere in the function is treated as local for the *entire* function, causing `UnboundLocalError` when read before that local assignment, not `NameError`.

## 4. Practice Questions with Answers

**Q1.** What does this print, and why?
```python
def mystery(a, b=[]):
    b.append(a)
    return b

print(mystery(1))
print(mystery(2))
```
**Answer:**
```
[1]
[1, 2]
```
The default list `b=[]` is created once at function-definition time and shared across calls that don't pass their own `b`. `mystery(1)` appends to it → `[1]`. `mystery(2)` appends to the *same* list → `[1, 2]`.

**Q2.** Write a function `describe(**kwargs)` that prints each keyword argument as `key: value`, one per line.

**Answer:**
```python
def describe(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

describe(name="Zoro", age=25)
# name: Zoro
# age: 25
```

**Q3.** What error (if any) does this raise, and why?
```python
total = 100
def spend(amount):
    total -= amount
    return total

spend(10)
```
**Answer:** `UnboundLocalError`. Because `total -= amount` assigns to `total` inside the function, Python treats `total` as local for the entire function body. The `-=` tries to read the local `total` before it has been assigned, raising `UnboundLocalError: local variable 'total' referenced before assignment`.

Fix:
```python
total = 100
def spend(amount):
    global total
    total -= amount
    return total

spend(10)
print(total)  # 90
```

**Q4.** Convert this function to a lambda:
```python
def is_even(n):
    return n % 2 == 0
```
**Answer:**
```python
is_even = lambda n: n % 2 == 0
print(is_even(4))  # True
```

**Q5.** Write a recursive function `sum_digits(n)` that returns the sum of the digits of a non-negative integer `n`. Identify the base case.

**Answer:**
```python
def sum_digits(n):
    if n <= 0:
        return 0
    return n % 10 + sum_digits(n // 10)

print(sum_digits(12345))  # 15
```
Base case: `n <= 0` → returns `0`. Each recursive call peels off the last digit (`n % 10`) and passes the remainder (`n // 10`) to the next call; the sum is accumulated as the calls return.

## 5. Problem Faced

Two reasoning gaps surfaced while answering the practice questions:
1. In Q1, the correct output was identified, but the explanation initially credited `append()` alone without identifying *why* the list already had a prior value on the second call.
2. In Q3, the correct error (`UnboundLocalError`) and fix (`global total`) were identified, but the underlying mechanism (compile-time scope resolution) was not initially explained.

## 6. Cause of the Problem

1. **Q1 cause**: Not yet internalized that default argument values are bound to the function object at **definition time**, not re-evaluated per call — so mutable defaults persist state across calls.
2. **Q3 cause**: Not yet internalized that Python determines whether a name is local or global by statically scanning the function body for assignment targets *before* the function ever runs — so any assignment to a name anywhere in the function makes it local throughout, even before that line executes.

## 7. Solution

1. For mutable default arguments: use `None` as the default and create the mutable object inside the function body:
   ```python
   def add_item(item, basket=None):
       if basket is None:
           basket = []
       basket.append(item)
       return basket
   ```
2. For scope/`UnboundLocalError` issues: use the `global` keyword when a global variable must be reassigned inside a function, or — as a generally preferred practice — avoid mutating globals altogether and instead return the new value, reassigning it at the call site.

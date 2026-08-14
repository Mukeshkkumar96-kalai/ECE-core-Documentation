# Phase 1: Programming Fundamentals

## 1. Today's Learning

- Python installation and running scripts (`python3 script.py`)
- Variables as names bound to objects (not typed memory slots)
- Core data types: `int`, `float`, `complex`, `str`, `bool`
- Type conversion (`int()`, `float()`, `str()`) and its failure cases
- Input/output basics (`input()`, `print()`, f-strings)
- All seven operator categories: arithmetic, comparison, logical, assignment, bitwise, membership, identity

---

## 2. Topic Explanation

### Setup & running programs
Python is interpreted — code runs line by line (technically compiled to bytecode first, but that's internal). Check your version with `python3 --version`; Python 2 is end-of-life (Jan 1, 2020), so avoid tutorials built on it. VS Code is just an editor — the Python extension is what adds linting/debugging.

### Variables & data types
```python
x = 5        # int
x = "hello"  # same name, now str — allowed, since names bind to objects
```
`bool` is a **subclass of `int`**: `True == 1` and `False == 0` are both `True`. This is used deliberately, e.g. `sum([True, False, True])` → `2`.

### Type conversion
```python
int("42")      # 42
float("3.14")  # 3.14
int(3.99)      # 3 — truncates, does not round
int("3.14")    # ValueError — string must look like a valid int literal
```

### Input/output
```python
name = input("Enter your name: ")   # input() ALWAYS returns str
age = int(input("Enter age: "))     # manual cast required for numbers
print(f"{name} is {age} years old") # f-strings preferred in modern Python
```

### Operators — all seven categories

| Category | Operators | Notes |
|---|---|---|
| Arithmetic | `+ - * / // % **` | `/` always returns float in Python 3; `//` floor division; `%` modulo |
| Comparison | `== != > < >= <=` | Return `bool` |
| Logical | `and or not` | Short-circuit; return an **operand**, not necessarily a bool |
| Assignment | `= += -= *= /= //= %= **= &= |= ^= >>= <<=` | Compound assignment |
| Bitwise | `& | ^ ~ << >>` | Operate on binary representation of integers |
| Membership | `in`, `not in` | Tests container membership |
| Identity | `is`, `is not` | Compares object identity (memory location), not value |

---

## 3. Misconceptions

1. **`int()` on a decimal-looking string works.** It does not — `int("3.14")` raises `ValueError`. You must go through `float()` first: `int(float("3.14"))`.
2. **`is` and `==` are interchangeable.** `is` checks object identity, not value equality. `a = 1000; b = 1000; a is b` can be `False` even though `a == b` is `True`, because Python doesn't guarantee caching of larger integers. Use `is` only for singletons like `x is None`.
3. **`5 is 5.0` is `False` because "one is int and one is float."** That's not the precise reason. `is` never checks type or value — it checks whether two names point to the *literal same object*. Different types are never the same object, but even same-value same-type objects aren't guaranteed identical (see point 2).
4. **`/` behaves like integer division if both operands are ints.** In Python 3, `/` always returns a `float` (`4 / 2` → `2.0`). Use `//` explicitly for floor division.

---

## 4. Practice Questions with Answers

**Q1.** What does `int(7.9)` return, and why?
**A:** `7`. `int()` on a float truncates toward zero — it does not round.

**Q2.** What is the output of `print(10 // 3, 10 % 3)`?
**A:** `3 1`. `//` gives floor division (10 ÷ 3 → 3), `%` gives the remainder (1).

**Q3.** What is the result of `True + True + False`?
**A:** `2`. `bool` is a subclass of `int`, so `True` = 1, `False` = 0; they sum arithmetically.

**Q4.** Does `5 is 5.0` evaluate to `True` or `False`? Explain using identity, not just type.
**A:** `False`. `is` compares object identity, not value. `5` (int) and `5.0` (float) are stored as separate, distinct objects — they can never be identical, regardless of value.

**Q5.** What error, if any, does `int("3.14")` raise, and how do you correctly convert `"3.14"` to an int?
**A:** Raises `ValueError` — the string isn't a valid integer literal. Correct approach: `int(float("3.14"))` → `3`.

---

## 5. Problem Faced

While solving the Phase 1 self-check exercise (`print(5 // 2, 5 % 2, 5 is 5.0, 5 == 5.0)`), the reasoning for why `5 is 5.0` is `False` was stated as *"False because 5 = int and 5.0 = float"*.

---

## 6. Cause of the Problem

The conclusion (`False`) was correct, but the justification conflated **type mismatch** with the actual mechanism. `is` does not evaluate type at all — it evaluates whether two names reference the same object in memory. Type mismatch is a side effect that guarantees non-identity here, not the rule `is` itself applies.

---

## 7. Solution

Reframe `is` strictly as an **identity check**: "are these the same object in memory?" — never "are these equal?" and never "are these the same type?".
- `5 is 5.0` → `False` because they're two distinct objects (an int object and a float object), full stop.
- Even `a = 1000; b = 1000; a is b` can be `False` despite matching type *and* value, because Python doesn't guarantee interning for larger integers.
- Correct rule of thumb: use `==` for value comparison, always. Reserve `is` exclusively for identity checks against singletons — primarily `is None`.

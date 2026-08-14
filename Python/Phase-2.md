# Phase 2: Control Flow

## 1. Today's Learning

- Conditional branching: `if` / `elif` / `else`
- `match-case` (Python 3.10+)
- `for` loop vs `while` loop, and `range()` boundaries
- Loop control: `break`, `continue`, `pass`
- Nested conditionals
- The `for...else` / `while...else` clause

---

## 2. Topic Explanation

### if / elif / else
```python
score = 72
if score >= 90:
    grade = "A"
elif score >= 75:
    grade = "B"
elif score >= 60:
    grade = "C"
else:
    grade = "F"
```
Python has no `switch` statement — `elif` chains (or `match-case`) are the idiomatic replacement. Indentation is the block delimiter, not cosmetic. There is no `elseif`/`else if` — only `elif`.

### match-case (3.10+)
```python
match status_code:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case 500 | 502 | 503:
        print("Server Error")
    case _:
        print("Unknown")
```
Does not exist before Python 3.10 — raises `SyntaxError` on older interpreters, no graceful fallback. `case _:` is the wildcard/default.

### for vs while
```python
for i in range(5):      # 0,1,2,3,4 — stop value excluded
    print(i)

n = 5
while n > 0:
    print(n)
    n -= 1               # no n-- in Python
```
`for` is a for-each over any iterable; `range()` simulates a counter loop. `range(0, 10, 2)` → 0,2,4,6,8 (start, stop, step).

### break, continue, pass
```python
for i in range(10):
    if i == 5:
        break        # exits the loop entirely
    if i % 2 == 0:
        continue     # skips rest of this iteration only
    print(i)
```
`pass` is a no-op placeholder — required wherever Python's syntax demands a statement but there's nothing to do. It does not skip or exit anything.

### for...else / while...else
```python
for i in range(5):
    if i == 10:
        break
else:
    print("loop finished without break")   # runs, since 10 never occurs
```
`else` here fires **only if the loop completed naturally** (no `break`). `continue` has no effect on whether `else` runs. Critically, the `else` binds to whichever loop it's indented to match — if it's indented under an inner loop, it belongs to that inner loop, not the outer one.

---

## 3. Misconceptions

1. **`for...else` means "otherwise," like an `if/else`.** It doesn't. It means "the loop completed without a `break`." `continue` firing has zero bearing on it.
2. **A `break` anywhere in the code disables the nearest `else`.** Only a `break` *inside that specific loop* disables *that loop's* `else`. Each loop's `else` is independent, tracked per-loop.
3. **`else` attached to `if` inside a loop behaves like `else` attached to the loop itself.** These are entirely different triggers that just share a keyword. `if/else` fires per-iteration on the condition; `for/else` fires once, only on natural loop completion. Indentation is what tells you which one you're looking at.
4. **`pass` skips the current branch/iteration.** It does nothing of the sort — it is a literal no-op. Skipping behavior is `continue`'s job, not `pass`'s.
5. **`n--` / `++n` work in Python.** They don't exist as syntax. Use `n -= 1` / `n += 1`.

---

## 4. Practice Questions with Answers

**Q1.**
```python
for i in range(4):
    if i == 2:
        break
    print(i, end=" ")
else:
    print("done")
```
**A:** Output: `0 1` (no "done"). `break` fires at `i == 2`, so the loop exits early — since it didn't complete naturally, the `else` clause is skipped entirely.

**Q2.**
```python
total = 0
for n in [1, 2, 3, 4, 5]:
    if n % 2 == 0:
        continue
    if n == 5:
        break
    total += n
print(total)
```
**A:** `total = 4`. `n=1` → adds 1 (total=1). `n=2` → even, `continue`, skips add. `n=3` → adds 3 (total=4). `n=4` → even, `continue`, skips add. `n=5` → hits `break` *before* `total += n` runs, so 5 is never added.

**Q3.**
```python
for i in range(2):
    for j in range(2):
        print(i, j)
    else:
        print("inner done")
print("outer finished")
```
**A:**
```
0 0
0 1
inner done
1 0
1 1
inner done
outer finished
```
The `else` is indented to match the **inner** `for j` loop, not the outer one — so it fires once per outer iteration (twice total), since the inner loop completes naturally both times with no `break` anywhere in the code.

**Q4.**
```python
n = 3
while n > 0:
    if n == 2:
        pass
    else:
        print(n)
    n -= 1
```
**A:** Output:
```
3
1
```
At `n=2`, the condition routes into `pass`, which does nothing (no print). `n=3` and `n=1` both hit the `else: print(n)` branch. Removing `pass` entirely (leaving the `if` block empty) would raise an `IndentationError` — `pass` exists to satisfy that syntax requirement while doing nothing at runtime.

**Q5.** Search `[4, 7, 2, 9, 5]` for the first number greater than 6; print it when found, print `"not found"` only if none exists — using `for...else`, no manual flag variable.
**A:**
```python
for n in [4, 7, 2, 9, 5]:
    if n > 6:
        print(n)
        break
else:
    print("not found")
```
Output: `7`. `n=4` fails the condition, continues. `n=7` passes → prints `7` → `break` exits immediately, so `2, 9, 5` are never checked. Because `break` fired, `else` is skipped. (`else` must be indented to match `for`, not `if` — that's what makes it a loop-level `else` instead of a conditional `else`.)

---

## 5. Problem Faced

Across the practice set, three distinct mistakes occurred:
- **Q3:** Predicted `"inner done"` would not print at all.
- **Q4:** Predicted the final printed value would be `2`.
- **Q5 (first attempt):** Wrote `else` indented under the `if` block instead of under the `for` block, and omitted `break` — causing the loop to check every element and print `"not found"` on every non-matching iteration instead of only at the end.

---

## 6. Cause of the Problem

- **Q3:** The `else`'s indentation (attached to the inner loop) was misread as belonging to the outer loop, or it was assumed a `break` was required somewhere for `else` to matter. Neither is correct — `else` fires on *natural completion of its own loop*, independent of any other loop.
- **Q4:** `pass` was treated as if it produced or forwarded some value (conflated with the final `n` after decrementing), rather than being read as a true no-op. The loop's actual printed values were confused with the loop counter's final state.
- **Q5:** `else` was indented under `if n > 6:` rather than under `for n in [...]:`, which silently converts a "loop-level else" into a "conditional else" — same keyword, different attachment point, different behavior. Combined with the missing `break`, this caused the loop to run to completion and re-evaluate the `else` condition on every iteration rather than once.

---

## 7. Solution

- **Read `else`'s trigger from its indentation, not its position in the file.** An `else` aligned with `for`/`while` is loop-level (fires once, only on no-`break` completion). An `else` aligned with `if` is conditional (fires per-iteration, whenever the condition is false). Same word, two unrelated mechanisms — indentation disambiguates them.
- **Treat `pass` as strictly inert.** It has no output, no side effect, no interaction with loop variables. If a branch should visibly do nothing, `pass` is correct; if you expect it to "produce" the skipped value, that expectation is wrong.
- **For "find first match" tasks, always pair `break` (to stop immediately after finding a match) with a loop-level `else` (to handle the "nothing found" case) — this replaces manual flag variables entirely** and was the corrected, working pattern in Q5's second attempt.

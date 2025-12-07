# Practical TDD: Draft First, Verify Later

**Stop staring at blank test files.**

## The Problem

Standard TDD demands you write a test before you write code. **But you cannot test a design that doesn't exist yet.**

If you are exploring a new idea, you often don't know the class names, the method signatures, or how the data flows. Trying to write a test first in this state leads to paralysis or bad tests that you have to rewrite five times.

## What TDD Actually Gives You

Before abandoning test-first, it's worth understanding what you'd lose:

- **Thorough regression safety** -- Every line of implementation has a test that would fail without it. This is stronger than code coverage, which only tells you a line *executed*. You can have 100% coverage and still delete a line without any test failing. With TDD, if it matters, removing it will cause a test to break.
- **Forced precision** — Writing the test first forces you to define "correct" before you code. You can't hand-wave; you have to commit to specific inputs and outputs.
- **Testability** -- Writing tests early helps make sure you think about testability early, making sure your tests are clear and straightforward instead of having to work around a poor design.
- **The red-green proof** — Seeing a test fail before it passes proves the test actually exercises the code you think it does. A test that's never been red might be testing nothing.

That last point is the critical one. Tests written after the fact often pass by accident—they may not actually cover the logic you intended. You never saw them fail, so you don't know if they *can* fail.

## The Solution

This pattern allows you to write the implementation first (Drafting) to figure out your design, then uses a simple toggle trick to provide many of the same benefits as standard red/green TDD.

**The goal is simple:** give you a clear boundary between code that is verified (code you've actually seen behave correctly) and code that is still just a sketch. Most of the friction around TDD comes from not knowing when it's "safe" to commit to an idea. This pattern gives you a small, repeatable way to turn uncertain exploratory code into code you can trust.

**How it preserves TDD's benefits:**
- **Thorough regression safety** -- Every fix is paired with a test that fails without it. No line of code survives without proving its necessity.
- **Forced precision** — You still write specific assertions before the fix is "live."
- **The red-green proof** — State III (fix off, test on) forces you to see the test fail. If it doesn't fail, your test isn't testing what you think it is.

## When to Use This

Don't use this if the test is obvious. If you already know exactly what the test code should look like, and you know the fix is easy, using these toggles will just feel like busywork. This tool is specifically for the moments when you don't yet know what test to write.

**Use this technique when you get stuck:**
- You don't know what API makes sense yet
- The design needs exploration before testing
- You're working with complex legacy code
- You need to think through architecture first

TDD wants you to add one small test at a time, but often, you cannot see the 'minimal step' until you have drafted the full solution. Drafting allows you to solve the architectural puzzle holistically. The verification phase then forces you to decompose that solution into atomic, proven steps.

You're not abandoning TDD, you're just allowing yourself a sketch phase, then forcing each piece of that sketch back through the classic "prove it fails, then make it pass" loop.

## What "Fix" Means

A **fix** is any change to production code that makes a new test pass—behavior that no existing test covers. From the test's perspective, bugs and missing features are the same thing: a gap between what the code does and what it should do. The code that closes that gap is a fix, whether you're repairing a defect or adding new functionality.

**This technique does NOT apply to:**
- **Changes to existing requirements:** The definition of 'Correct' has changed. In that case, treat the updates to the test and the code as a single atomic change. You can modify them in whatever order feels best to you.
- **Refactoring:** You want to restructure the code to make it more maintainable without adding complexity. Tests should always be passing during these changes and add no new complexity.

## The Technique

1. Draft both pieces (in any order, any level of detail, ideally focusing on a single, cohesive unit of behavior or a small set of related changes)
   - Write the code you think will work (commented out or behind a flag—examples below show when to use each)
   - Write the tests that you think would verify it (commented out or behind a flag)
   - Think through the design. Revise. Explore. This is just sketching.

2. **Pick one minimal test/fix pair** from your draft
   - Look at one line of code that you've drafted. Ask, "What is the simplest example that wouldn't work without it?"
   - Minimal: the simplest (fewest steps and code paths) test/fix where you can still verify all 4 states

   **Finding minimal pairs:**
   Pick one line from your draft that you want to verify. Think about why it's there. What's an example of something that wouldn't work properly without it? That example is your test.

3. **Choose an activation mechanism**
   - Decide how you are going to turn the fix and test off and on.
   - This can be
     - Commenting out the code to turn it off, uncommenting it to turn it on
     - Using a preprocessor flag that guards the change and turning it on or off
     - Using test framework options, like a "skip" flag for a test.
   - Use whatever approach makes sense to you for switching the fix and the test off and on.

4. **Verify it through 4 states:**
   |                | **Test off**               | **Test on**                |
   |----------------|----------------------------|----------------------------|
   | **Fix off**    | **State I** (tests pass)   | **State III** (tests fail) |
   | **Fix on**     | **State II** (tests pass)  | **State IV** (tests pass)  |

   - Use this same test scope for all 4 states.
   - Run all tests for the module you're changing (not just the new test).
   - Verify that the tests pass or fail as indicated for each state.
   - These states can be verified in any order, but the given order tends to work well.

   - **State I:** Both off → Must be green (no pre-existing failures)
     - *This is your starting point. If tests aren't green before you begin, fix that first.*
   - **State II:** Fix on, test off → Must be green (no regressions)
     - *A quick sanity check: does your fix break anything that was already working?*
     - *Note: Traditional TDD verifies this implicitly. Making it explicit helps isolate issues when State IV fails—did your fix break existing tests, or does your new test have problems? This catches fixes that are overly broad—changing more behavior than intended.*
   - **State III:** Fix off, test on → Must be red (test detects the missing behavior)
   - **State IV:** Both on → Must be green (fix makes test pass)

   **If something fails:**
   - State I red → Fix pre-existing failures
   - State II red → Your fix breaks existing tests, revise it
   - State III green → Your test doesn't actually test the fix, revise it
   - State IV red → Your fix doesn't work, revise it
   - *Any revision means you must re-verify all states with the new test/fix pair*

5. **Remove the scaffolding**
    - If you used flags, bake them in and simplify.
    - If this requires any code changes, run the tests again to double-check.

6. **Repeat** for the next test/fix pair

---

## Example 1: Single Location (The "Comment" Toggle)

The "Draft" phase is just writing code in comments. The "Verification" phase is just deleting the `#` to let the compiler see it.

### 1. The Draft (Safe Mode)
Write your implementation and test freely. Keep them commented out so they don't break the build yet.

```python
# Baseline: The code does not exist yet.
#
# def calculate_discount(price, is_member):
#     if is_member:
#         return price * 0.9
#     return price
#
# def test_discount():
#     assert calculate_discount(100, True) == 90
```

### 2. Verify State I: Both Off (Green)
Run your existing tests with everything still commented out. This confirms you're starting from a clean baseline.

### 3. Verify State II: Fix On, Test Off (Green)
Uncomment **only the fix**. Run tests—they should still pass. This confirms your new code doesn't break anything existing.

```diff
- # def calculate_discount(price, is_member):
+ def calculate_discount(price, is_member):
- #     if is_member:
+     if is_member:
- #         return price * 0.9
+         return price * 0.9
- #     return price
+     return price

  # def test_discount():
  #     assert calculate_discount(100, True) == 90
```

### 4. Verify State III: Fix Off, Test On (Red)
Comment the fix back out, uncomment **only the test**. The test must fail—this proves it actually detects the missing behavior.

```diff
  # def calculate_discount(price, is_member):
  #     if is_member:
  #         return price * 0.9
  #     return price

- # def test_discount():
+ def test_discount():
- #     assert calculate_discount(100, True) == 90
+     assert calculate_discount(100, True) == 90
```

### 5. Verify State IV: Both On (Green)
Uncomment **the fix**. Now both are live and the test passes.

```diff
- # def calculate_discount(price, is_member):
+ def calculate_discount(price, is_member):
- #     if is_member:
+     if is_member:
- #         return price * 0.9
+         return price * 0.9
- #     return price
+     return price

  def test_discount():
      assert calculate_discount(100, True) == 90
```

## Example 2: Multi-Location (The "Flag" Toggle)

When your draft touches multiple files, comments are messy. Instead, use a temporary "Toggle Block" at the top of your file (or in a config).

### 1. The Draft (Safe Mode)
Write the code you *want* to have, guarded by a flag.

```python
# --- TEMPORARY SCAFFOLDING ---
FIX_SILVER = False
TEST_SILVER = False
# -----------------------------

# Location A: The Logic
def get_tier(is_member):
    if FIX_SILVER and is_member:
        return "silver"
    return None

# Location B: The Test
def test_tiers():
    assert get_tier(False) is None
    if TEST_SILVER:
        assert get_tier(True) == "silver"
```

### 2. Verify: Watch it Fail (State III)
Flip the **Test Switch** to `True`. The logic is still disabled, so the test must fail.

```diff
  # --- TEMPORARY SCAFFOLDING ---
  FIX_SILVER = False
- TEST_SILVER = False
+ TEST_SILVER = True
  # -----------------------------
```

### 3. Verify: Watch it Pass (State IV)
Flip the **Fix Switch** to `True`. Now the logic is live and the test passes.

```diff
  # --- TEMPORARY SCAFFOLDING ---
- FIX_SILVER = False
+ FIX_SILVER = True
  TEST_SILVER = True
  # -----------------------------
```

### 4. Cleanup
Delete the flags and the `if` guards. You are done.

```python
def get_tier(is_member):
    if is_member:
        return "silver"
    return None

def test_tiers():
    assert get_tier(False) is None
    assert get_tier(True) == "silver"
```

## Why This Works

- **Low friction:** You don't have to commit to a design before you understand the problem
- **High confidence:** Code that survives all 4 states has been tested from multiple angles—you've seen it both fail and succeed
- **No waste:** You keep only the tests that prove something
- **Safe refactoring:** Once verified, you can restructure freely knowing the behavior is locked in

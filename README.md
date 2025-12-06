# TDD for People Who Think TDD is Impractical

Get the confidence of TDD, even when the design is still unclear.

## A common complaint about TDD…

A lot of people find TDD confusing or impractical:

    “How am I supposed to test something that doesn’t exist?”
    “How do I know what test to write when I haven’t figured out the design?”
    “If the design changes while I'm exploring, won’t that just break tests constantly?”

This pattern is meant for that fuzzy phase — the part where you’re still figuring things out and the classic “write the test first” rule feels unrealistic.

**The goal of this approach is simple:**
to give you a clear boundary between code that is verified — code you’ve actually seen behave correctly — and code that is still just a sketch or experiment.
Most of the friction around TDD comes from not knowing when it’s “safe” to commit to an idea.
This pattern gives you a small, repeatable way to turn uncertain exploratory code into code you can trust.

## What "Fix" Means

One small idea we need before we go further

Before the technique makes sense, we need to name one thing clearly.

What a “fix” means

A fix is any change to production code that adds complexity. By complexity, I mean additional steps or additional code paths. It might be repairing a bug where we didn't handle an edge case before, or it might be adding new behavior to satisfy requirements. Either way, it is behavior that we had no existing test for. If we're starting from a baseline where all current complexity has been shown to be necessary through existing tests, then a fix is necessarily something that adds complexity.


**This technique does NOT apply to:**
- **Changes to existing requirements:** The definition of 'Correct' has changed. In that case, treat the updates to the test and the code as a single atomic change. You can modify them in whatever order feels best to you.
- **Refactoring:** You want to restructure the code to make it more maintainable without adding complexity. Tests should always be passing during these changes and add no new complexity.

## When to Use This


Don't use this if the test is obvious. If you already know exactly what the test code should look like, and you know the fix is easy, using these toggles will just feel like busywork.

Use this when you are staring at a blank screen. This tool is specifically for the moments when you don't know what test to write.

**Use this technique when you get stuck:**
- You don't know what API makes sense yet
- The design needs exploration before testing
- You're working with complex legacy code
- You need to think through architecture first

Often, you cannot see the 'minimal step' until you have drafted the full solution. Drafting allows you to solve the architectural puzzle holistically. The verification phase then forces you to decompose that solution into atomic, proven steps.

You’re not abandoning TDD, you’re just allowing yourself a sketch phase, then forcing each piece of that sketch back through the classic “prove it fails, then make it pass” loop.

## The Technique

1. Draft both pieces (in any order, any level of detail, ideally focusing on a single, cohesive unit of behavior or a small set of related changes)
   - Write the code you think will work (commented out or behind a flag—examples below show when to use each)
   - Write the tests that you think would verify it (commented out or behind a flag)
   - Think through the design. Revise. Explore. This is just sketching.

2. **Pick one minimal test/fix pair** from your draft
   - Minimal: the simplest (fewest steps and code paths) test/fix where you can still verify all 4 states
   - The "fix" is production code that requires a new test to verify it

3. **Choose an activation mechanism**
   - Decide how you are going to turn the fix and test off and on.
   - This can be
     - Commenting out the code to turn it off, uncommenting it to turn it on
     - Using a preprocessor flag that guards the change and turning it on or off
     - Using test framework options, like a "skip" flag for a test.
   - Use whatever approach makes sense to you.

4. **Verify it through 4 states:**
   |                | **Test off**               | **Test on**                |
   |----------------|----------------------------|----------------------------|
   | **Fix off**    | **State I** (tests pass)   | **State III** (tests fail) |
   | **Fix on**     | **State II** (tests pass)  | **State IV** (tests pass)  |

   - **Run all tests** for the module you're changing (not just the new test)
   - Use this same test scope for all 4 states
   - Verify by toggling on/off
   - These states can be verified in any order, but the given order tends to work well.

   - **State I:** Both off → Must be green (no pre-existing failures)
   - **State II:** Fix on, test off → Must be green (no regressions)
     - *Note: Traditional TDD verifies this implicitly. Making it explicit helps isolate issues when State IV fails—did your fix break existing tests, or does your new test have problems?*
   - **State III:** Fix off, test on → Must be red (test detects the missing behavior)
   - **State IV:** Both on → Must be green (fix makes test pass)

   **If something fails:**
   - State I red → Fix pre-existing failures first
   - State II red → Your fix breaks existing tests, revise it
   - State III green → Your test doesn't actually test the fix, revise it
   - State IV red → Your fix doesn't work, revise it
   - *Any revision means you must re-verify all states with the new test/fix pair*

5. **Remove the scaffolding** (uncomment permanently, delete the flags)
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

### 2. Verify: Watch it Fail (State III)
Uncomment **only the test**. This proves the test fails (Red) and confirms you aren't getting a false positive.

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

### 3. Verify: Watch it Pass (State IV)
Uncomment **the fix**. This proves the code works (Green).

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

- **Design freedom:** Draft in any order, explore the API, think ahead
- **TDD discipline maintained:** Every new test must fail without the fix (State III)
- **Features and bugs treated equally:** Both require a new test that demonstrates missing behavior
- **No false confidence:** State III proves your test can actually fail
- **Verified fixes:** State IV proves your fix actually works and doesn't break anything
- **No waste:** You keep only tests that demonstrated they can catch real problems

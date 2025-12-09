# Practical TDD: Draft Holistically, Verify Atomically

**Stop staring at blank test files.**

## The Problem

Standard TDD demands you write a test before you write code. **But you cannot test a design that doesn't exist yet.**

If you are exploring a new idea, you often don't know the class names, the method signatures, or how the data flows. Trying to write a test first in this state leads to paralysis or bad tests that you have to rewrite five times.

## What TDD Actually Gives You

Before abandoning TDD, it's worth understanding what you'd lose:

- **Thorough regression safety** -- Every line of implementation has a test that would fail without it. This is stronger than code coverage, which only tells you a line *executed*. You can have 100% coverage and still delete a line without any test failing. With TDD, if any part of the implementation matters, removing it will cause a test to break. This follows Google's "Beyoncé Rule": "If you liked it, then you should have put a test on it." (as popularized in *Software Engineering at Google*).
- **No unnecessary code** -- Every line of the implementation has an example to go with it. If you can't point to an example that requires the extra complexity, you don't write it.
- **Perspective** -- Thinking about examples of how your implementation will be used early on helps make sure you stay focused on results.
- **Testability** -- Thinking about examples early helps you think about testability early, making sure your tests are clear and straightforward instead of having to work around a poor design.
- **Verified Examples** -- You don't just write the examples. You show that each example actually demonstrates a problem when the implementation is insufficient. You can always see why a part of an implementation exists by temporarily removing it and seeing the example that fails.

That last point is the critical one. Tests written after the fact often pass by accident -- they may not actually cover the logic you intended. You never saw them fail, so you don't know if they *can* fail.

## The Solution

This pattern allows you to write the implementation first (Drafting) to figure out your design, then uses a simple toggle trick to provide many of the same benefits as strictly writing the examples first.

**The goal is simple:** to give you a clear boundary between code that is verified (code you've actually seen behave correctly) and code that is still just a sketch. Most of the friction around TDD comes from not knowing when it's "safe" to commit to an idea. This pattern gives you a small, repeatable way to turn uncertain exploratory code into code you can trust.

**How it preserves TDD's benefits:**
- **Thorough regression safety** -- Every fix is paired with a test that fails without it. No line of code survives without proving its necessity, thereby adhering to the Beyoncé Rule.
- **No unnecessary code** -- You still only write code to satisfy tests. The draft is just a sketch; verified code still requires an example.
- **Perspective** -- You still write examples before the fix is "live," keeping you focused on results.
- **Testability** -- You still bring in tests early, while your design is still easy to change.
- **Verified Examples** -- State III (fix off, test on) forces you to see the test fail. If it doesn't fail, your example isn't demonstrating what you think it is.

This techinque is really just expanding on [TDD's three laws](http://butunclebob.com/ArticleS.UncleBob.TheThreeRulesOfTdd). We clarify that drafting and experimentation isn't "writing code" in a way that would matter to TDD.

*Note: Kent Beck's original TDD formulation emphasizes listing tests upfront but cautions against sketching implementation too early: "If you need an implementation sketch in Sharpie on a napkin, go ahead, but you might not really need it." That's valuable advice—but when you're stuck, drafting the implementation is often the quickest way to break through test paralysis. Don't let his advice push you towards abandoning TDD altogether. See [Canon TDD](https://tidyfirst.substack.com/p/canon-tdd).*

## When to Use This

Don't use this if the test is obvious. If you already know exactly what the test code should look like, and you know the fix is easy, using these toggles will just feel like busywork. This tool is specifically for the moments when you don't yet know what test to write.

**Use this technique when you get stuck:**
- You don't know what API makes sense yet
- The design needs exploration before testing
- You're working with complex legacy code
- You need to think through architecture first

TDD wants you to add one small test at a time, but often, you cannot see a good test to add until you have drafted the full solution. Drafting allows you to solve the architectural puzzle holistically. The verification phase then forces you to decompose that solution into atomic, proven steps.

You're not abandoning TDD, you're just allowing yourself a sketch phase, then forcing each piece of that sketch back through the classic "prove it fails, then make it pass" loop.

## Terminology

In TDD, all behavior exists because you have an example of why you need it. You have the fewest examples that fully demonstrate the desired behavior, and you have the simplest implementation that satisfies these examples. We break the examples and implementation into tiny pieces called tests and fixes:

**Test:** An example of the desired behavior
**Fix:** An implementation of the desired behavior

Thinking of all new behavior as fixes keeps you honest: if you don't have an example of why you need some part of the implementation, then why are you writing it?

**This does NOT apply to:**
- **Changes to existing requirements:** If the definition of "correct" has changed, treat the updates to the test and the code as a single atomic change.
- **Refactoring:** If you're restructuring code without changing behavior, you aren't adding new tests, and all existing tests should stay green throughout.

## The Technique

1. **Draft your implementation and your tests**
   - Do your drafting in any order and level of detail that helps you understand what you are trying to solve.
   - Sketch the logic you think will work — this can be pseudocode, comments, notes, or real code
   - Sketch the tests you think would verify it — same flexibility applies
   - Think through the design. Revise. Explore. Make sure it is clear that what you are writing is not the final production code.

2. **Pick one line from your draft**
   - Look at a line you wrote. Ask: "What is this trying to do? What's the simplest example that would fail without it?"
   - That example is your test. The code to make it pass is your fix.

3. **Choose an activation mechanism**
   - Decide how you are going to turn the fix and test off and on.
   - This can be
     - Commenting out the code to turn it off, uncommenting it to turn it on
     - Using a preprocessor flag that guards the change and turning it on or off
     - Using test framework options, like a "skip" flag for a test.
   - Use whatever approach makes sense to you for switching the fix and the test off and on.

4. **Use a 4-state verification matrix to build up a test/fix pair**

   |                | **Test off**               | **Test on**                |
   |----------------|----------------------------|----------------------------|
   | **Fix off**    | **State I** (tests pass)   | **State III** (tests fail) |
   | **Fix on**     | **State II** (tests pass)  | **State IV** (tests pass)  |

   Bounce between these states to verify them, until they are all verified with the same test/fix pair:
   - **State I** (both off, must be green): Stay here while writing inactive code or refactoring — confirms you aren't changing existing behavior.
   - **State II** (fix on, test off, must be green): Stay here while building the fix. This confirms you're not breaking existing tests. This also catches fixes that are overly broad, changing more behavior than intended. If you change the fix, you need to re-verify state IV if you've verified it already.
   - **State III** (fix off, test on, must be red): Build test code here until the test fails as expected. This confirms the test actually detects missing behavior. If you change the test here, you need to re-verify state IV if you've verified it already.
   - **State IV** (both on, must be green): Adjust the test and the fix here until the tests pass. If you change the test, you need to re-verify state III if you've verified it already.

   You only re-verify a state if you've changed what it depends on: State II depends on the fix, State III depends on the test, State IV depends on both.

5. **Remove the scaffolding**
    - If you used flags, bake them in and simplify.
    - If this requires any code changes, run the tests again to double-check.

6. **Repeat** for the next test/fix pair

---

*Note: In practice, these are tiny edits and quick toggles—seconds each, not minutes. The examples below are verbose because static documentation can't show the rapid, iterative rhythm of the actual workflow.*

## Example 1: Building Incrementally

### 1. The Draft

Sketch out your approach — a rough algorithm, not necessarily compilable code.

```
# calculate_discount(price, is_member)
# if not a member, just return the price
# if a member, take 10% off and return that

# tests:
# non-member paying 100 should pay 100
# member paying 100 should pay 90
```

This is your target. You'll build toward it one verified step at a time.

### 2. First Test/Fix Pair: Non-member path

Start with a stub that crashes on any unhandled path:

```python
def calculate_discount(price, is_member):
    assert False

def test_calculate_discount():
    pass
```

Look at your draft. "If not a member, just return the price" — that's a path you can implement.

**Enter State III** (fix off, test on). Objective: make the test fail.

Build up the test — just enough to hit the assert:

```diff
 def test_calculate_discount():
-    pass
+    calculate_discount(100, False)
```

Run it. Crashes on `assert False`. State III objective met.

**Enter State IV** (fix on, test on). Objective: make the test pass.

Build up the fix — just enough to not crash:

```diff
 def calculate_discount(price, is_member):
+    if not is_member:
+        return price
     assert False
```

Run it. Passes.

**Enter State III.** Objective: make the test fail.

```diff
 def calculate_discount(price, is_member):
-    if not is_member:
-        return price
+    # if not is_member:
+    #     return price
     assert False
```

The test still passes. The test isn't verifying behavior yet. Expand the test:

```diff
 def test_calculate_discount():
-    calculate_discount(100, False)
+    assert calculate_discount(100, False) == 100
```

Run it. Fails as expected.

**Enter State IV.** Objective: make the test pass.

```diff
 def calculate_discount(price, is_member):
-    # if not is_member:
-    #     return price
+    if not is_member:
+        return price
     assert False
```

Run it. Passes as expected.

First path verified.

### 3. Second Test/Fix Pair: Member path

Look at the draft. "If a member, take 10% off" — next path.

**Enter State III.** Objective: make the test fail.

Add to the test — just enough to hit the assert:

```diff
 def test_calculate_discount():
     assert calculate_discount(100, False) == 100
+    calculate_discount(100, True)
```

Run it. Crashes on `assert False`. State III objective met.

**Enter State IV.** Objective: make the test pass.

Build up the fix:

```diff
 def calculate_discount(price, is_member):
     if not is_member:
         return price
-    assert False
+    return price * 0.9
```

Run it. Passes.


**Enter State III.** Objective: make the test fail

```diff
 def calculate_discount(price, is_member):
     if not is_member:
         return price
-    return price * 0.9
+    # return price * 0.9
```

The test still passes. The test isn't verifying behavior yet.

Expand the test:

```diff
 def test_calculate_discount():
     assert calculate_discount(100, False) == 100
-    calculate_discount(100, True)
+    assert calculate_discount(100, True) == 90
```

Run it. Fails as expected.

**Enter State IV.** Objective: make the test pass.

```diff
 def calculate_discount(price, is_member):
     if not is_member:
         return price
-    # return price * 0.9
+    return price * 0.9
```

Run it. Passes as expected.


Second path verified.

### 4. Refactor

Both paths verified. Simplify:

```diff
 def calculate_discount(price, is_member):
-    if not is_member:
-        return price
-    return price * 0.9
+    return price * 0.9 if is_member else price
```

Tests stay green.

## Example 2: Multi-Location (The "Flag" Toggle)

When your fix touches multiple locations, commenting and uncommenting becomes error-prone. Use flags — flip a single variable to toggle all the pieces at once.

### 1. The Baseline

A system with silver tier discounts already exists:

```python
def _get_member_tier(is_member):
    if not is_member:
        return None
    return "silver"

def _calculate_discount(price, tier):
    if tier == "silver": return price * 0.9
    return price

def process_purchase(price, is_member):
    tier = _get_member_tier(is_member)
    return _calculate_discount(price, tier)

def test_tiered_discounts():
    assert process_purchase(100, False) == 100
    assert process_purchase(100, True) == 90   # silver
```

### 2. The Draft

Now we're adding gold tier for members with 5+ years. Write the code you want, guarded by flags:

```python
# --- TEMPORARY SCAFFOLDING ---
FIX_GOLD = False
TEST_GOLD = False
# -----------------------------

# Location A: Tier logic (private)
def _get_member_tier(is_member, years):
    if not is_member:
        return None
    if FIX_GOLD:
        if years >= 5: return "gold"
    return "silver"

# Location B: Discount logic (private)
def _calculate_discount(price, tier):
    if FIX_GOLD:
        if tier == "gold": return price * 0.8
    if tier == "silver": return price * 0.9
    return price

# Public API
def process_purchase(price, is_member, years):
    tier = _get_member_tier(is_member, years)
    return _calculate_discount(price, tier)

# Location C: The test
def test_tiered_discounts():
    assert process_purchase(100, False, 0) == 100
    assert process_purchase(100, True, 3) == 90   # silver
    if TEST_GOLD:
        assert process_purchase(100, True, 5) == 80   # gold
```

### 3. Enter State III (fix off, test on)

Objective: make the test fail.

```diff
 # --- TEMPORARY SCAFFOLDING ---
 FIX_GOLD = False
-TEST_GOLD = False
+TEST_GOLD = True
 # -----------------------------
```

Run it. Fails as expected.

### 4. Enter State IV (fix on, test on)

Objective: make the test pass.

```diff
 # --- TEMPORARY SCAFFOLDING ---
-FIX_GOLD = False
+FIX_GOLD = True
 TEST_GOLD = True
 # -----------------------------
```

Run it. Passes as expected.

### 5. Cleanup

Remove flags and guards:

```diff
-# --- TEMPORARY SCAFFOLDING ---
-FIX_GOLD = True
-TEST_GOLD = True
-# -----------------------------

 def _get_member_tier(is_member, years):
     if not is_member:
         return None
-    if FIX_GOLD:
-        if years >= 5: return "gold"
-    return "silver"
+    if years >= 5: return "gold"
+    return "silver"

 def _calculate_discount(price, tier):
-    if FIX_GOLD:
-        if tier == "gold": return price * 0.8
+    if tier == "gold": return price * 0.8
     if tier == "silver": return price * 0.9
     return price

 def process_purchase(price, is_member, years):
     tier = _get_member_tier(is_member, years)
     return _calculate_discount(price, tier)

 def test_tiered_discounts():
     assert process_purchase(100, False, 0) == 100
     assert process_purchase(100, True, 3) == 90   # silver
-    if TEST_GOLD:
-        assert process_purchase(100, True, 5) == 80   # gold
+    assert process_purchase(100, True, 5) == 80   # gold
```

Tests stay green.

## Why This Works

- **Low friction:** You don't have to commit to a design before you understand the problem
- **High confidence:** Code that survives all 4 states has been tested from multiple angles -- you've seen it both fail and succeed
- **No waste:** You keep only the tests that prove something
- **Safe refactoring:** Once verified, you can restructure freely knowing the behavior is locked in

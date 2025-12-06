# TDD for People Who Think TDD is Impractical

**The Problem:** You can't write the test first because you don't know what the API should look like yet.

**The Solution:** Design thinking and test verification are separate activities.

## What "Fix" Means

> **In this guide: "fix" = any production code change that requires a new test to verify it**

In TDD, new features and bug fixes are treated identically—both require adding a new test:

- **New feature:** The test didn't exist because the feature didn't exist
- **Bug fix:** The test didn't exist, or it would have caught the bug.

Both cases add complexity to the production code that must be justified by a new failing test. This technique maintains that discipline even when you draft both pieces first.

**This technique does NOT apply to:**
- **Requirement changes:** The definition of 'Correct' has changed. Treat the updates to the test and the code as a single atomic change. You can modify them in whatever order feels best.
- **Refactoring:** Change code/test structure while keeping test expectations unchanged and green and not adding complexity.

## When to Use This

**Start with traditional TDD** (write test → red → write fix → green). It's the fastest path when it works.

**Use this technique when you get stuck:**
- You don't know what API makes sense yet
- The design needs exploration before testing
- You're working with complex legacy code
- You need to think through architecture first

Often, you cannot see the 'minimal step' until you have drafted the full solution. Drafting allows you to solve the architectural puzzle holistically. The verification phase then forces you to decompose that solution into atomic, proven steps.

## The Technique

1. Draft both pieces (in any order, any level of detail, ideally focusing on a single, cohesive unit of behavior or a small set of related changes)
   - Write the code you think will work (commented out or behind a flag—examples below show when to use each)
   - Write the tests that you think would verify it (commented out or behind a flag)
   - Think through the design. Revise. Explore. This is just sketching.

2. **Pick one minimal test/fix pair** from your draft
   - Minimal: the simplest (fewest steps and code paths) test/fix where you can still verify all 4 states
   - The "fix" is production code that requires a new test to verify it

3. **Verify it through 4 states:**
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

4. **Remove the scaffolding** (uncomment permanently, delete the flags)
    - If this requires any code changes, run the tests again to double-check.

6. **Repeat** for the next test/fix pair

## Example 1: Single Location (Comments)

(✓ indicates the state was verified by running tests)

```python
# Draft (both commented out - just thinking)
# def calculate_discount(price, is_member):
#     if is_member:
#         return price * 0.9
#     return price
#
# def test_discount():
#     assert calculate_discount(100, False) == 100
#     assert calculate_discount(100, True) == 90

# First pair: non-member case
# State I: Both off → Green ✓
# State II: Test off, Fix on - Green ✓
def test_discount():
    pass
#   assert calculate_discount(100, False) == 100
#   assert calculate_discount(100, True) == 90
def calculate_discount(price, is_member):
    return price
# State III: Test on, fix off → Red ✓
def test_discount():
    assert calculate_discount(100, False) == 100
#   assert calculate_discount(100, True) == 90
# def calculate_discount(price, is_member):
#     return price
# State IV: Both on → Green ✓
def test_discount():
    assert calculate_discount(100, False) == 100
#   assert calculate_discount(100, True) == 90
def calculate_discount(price, is_member):
    return price

# Second pair: member discount
# State I: Test off, Fix off → Green ✓
def test_discount():
    assert calculate_discount(100, False) == 100
    # assert calculate_discount(100, True) == 90
def calculate_discount(price, is_member):
#     if is_member:
#         return price * 0.9
    return price
# State II: Test off, Fix on  → Green ✓
def test_discount():
    assert calculate_discount(100, False) == 100
    # assert calculate_discount(100, True) == 90
def calculate_discount(price, is_member):
    if is_member:
        return price * 0.9
    return price
# State III: New test on, new fix off → Red ✓
def test_discount():
    assert calculate_discount(100, False) == 100
    assert calculate_discount(100, True) == 90
def calculate_discount(price, is_member):
#     if is_member:
#         return price * 0.9
    return price
# State IV: Both on → Green ✓
def test_discount():
    assert calculate_discount(100, False) == 100
    assert calculate_discount(100, True) == 90
def calculate_discount(price, is_member):
    if is_member:
        return price * 0.9
    return price
```

## Example 2: Multi-Location (Centralized Toggles)

When your fix requires changes in **multiple locations**, use a centralized toggle. This ensures all changes activate atomically and allows the new API structure to exist (preventing crashes) while the logic is disabled.

```python
# Baseline: Basic discount system exists
# Draft: Adding tiered member discounts (silver and gold)

# First pair: Silver tier (minimal - changes 3 locations atomically)
FIX_SILVER = False
TEST_SILVER = False

# Location 1: Membership tier detection
def get_member_tier(is_member, years):
    if FIX_SILVER:
        if not is_member:
            return None
        return "silver"  # Minimal: only silver for now
    return None

# Location 2: Tier-based discount calculation
def calculate_discount(price, tier):
    if FIX_SILVER:
        if tier == "silver":
            return price * 0.9
    return price

# Location 3: Main purchase flow
def process_purchase(price, is_member, years):
    if FIX_SILVER:
        tier = get_member_tier(is_member, years)
        return calculate_discount(price, tier)
    return price

def test_tiered_discounts():
    assert process_purchase(100, False, 0) == 100  # Existing
    if TEST_SILVER:  # One new assertion
        assert process_purchase(100, True, 3) == 90

# Verify first pair through 4 states, then remove scaffolding

# Second pair: Gold tier (builds on silver baseline)
FIX_GOLD = False
TEST_GOLD = False

def get_member_tier(is_member, years):
    if not is_member:
        return None
    if FIX_GOLD:  # New logic
        return "gold" if years >= 5 else "silver"
    return "silver"  # Existing: silver only

def calculate_discount(price, tier):
    if FIX_GOLD:  # New logic
        if tier == "gold":
            return price * 0.8
    if tier == "silver":  # Existing
        return price * 0.9
    return price

def test_tiered_discounts():
    assert process_purchase(100, False, 0) == 100
    assert process_purchase(100, True, 3) == 90  # Existing
    if TEST_GOLD:  # One new assertion
        assert process_purchase(100, True, 5) == 80

# Verify second pair through 4 states, then remove scaffolding

# Final: All scaffolding removed
def get_member_tier(is_member, years):
    if not is_member:
        return None
    return "gold" if years >= 5 else "silver"

def calculate_discount(price, tier):
    if tier == "gold":
        return price * 0.8
    elif tier == "silver":
        return price * 0.9
    return price

def process_purchase(price, is_member, years):
    tier = get_member_tier(is_member, years)
    return calculate_discount(price, tier)

def test_tiered_discounts():
    assert process_purchase(100, False, 0) == 100
    assert process_purchase(100, True, 3) == 90
    assert process_purchase(100, True, 5) == 80
```

## Why This Works

- **Design freedom:** Draft in any order, explore the API, think ahead
- **TDD discipline maintained:** Every new test must fail without the fix (State III)
- **Features and bugs treated equally:** Both require a new test that demonstrates missing behavior
- **No false confidence:** State III proves your test can actually fail
- **Verified fixes:** State IV proves your fix actually works and doesn't break anything
- **No waste:** You keep only tests that demonstrated they can catch real problems

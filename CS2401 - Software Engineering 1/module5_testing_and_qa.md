# Module 5: Software Testing and Quality Assurance

**Course:** CS 2401 – Software Engineering 1
**Module:** 5 of 8

---

## Learning Objectives

By the end of this module, students will be able to:
- Explain the different levels of software testing
- Distinguish between black-box and white-box testing
- Write unit tests and understand test coverage
- Describe the test lifecycle and defect management
- Understand automated testing and CI pipelines

---

## 5.1 Why Testing Matters

Testing is not about proving software works — it is about **finding defects before users do**.

> "Testing shows the presence, not the absence, of bugs." — Edsger Dijkstra

### The Cost of Late Bug Fixes

The later a bug is found, the more expensive it is to fix:

```
Phase Where Bug Is Found    │  Relative Cost to Fix
────────────────────────────┼───────────────────────
Requirements                │  1×
Design                      │  5×
Development                 │  10×
Testing                     │  20×
Production                  │  100×
```

**Real Example:** The Knight Capital Group (2012) deployed faulty trading software that went undetected until it went live. In 45 minutes, the system executed millions of erroneous trades, losing the company **$440 million**.

---

## 5.2 Testing Terminology

| Term | Definition |
|---|---|
| **Error** | A human mistake during development |
| **Defect / Bug** | A flaw in the code resulting from an error |
| **Failure** | Observable incorrect behavior of the system |
| **Test Case** | A specific set of inputs, conditions, and expected results |
| **Test Suite** | A collection of test cases |
| **Test Coverage** | Percentage of code exercised by tests |

---

## 5.3 Levels of Testing

### The Testing Pyramid

```
           ┌──────────────┐
           │   E2E Tests  │  ← Few, slow, expensive
           │  (UI/System) │
          ─┴──────────────┴─
         ┌──────────────────┐
         │ Integration Tests│  ← Some, medium speed
        ─┴──────────────────┴─
       ┌──────────────────────┐
       │      Unit Tests      │  ← Many, fast, cheap
       └──────────────────────┘
```

Aim for most tests at the bottom (unit) and fewer at the top (E2E).

---

### Level 1: Unit Testing

Tests **individual functions or methods** in isolation.

**Characteristics:**
- Fast (milliseconds per test)
- Written by developers
- Uses mocking/stubbing for dependencies

**Example (Python with pytest):**

```python
# Function under test
def calculate_discount(price, discount_pct):
    if discount_pct < 0 or discount_pct > 100:
        raise ValueError("Discount must be between 0 and 100")
    return price * (1 - discount_pct / 100)

# Unit tests
def test_calculate_discount_normal():
    assert calculate_discount(100, 20) == 80.0

def test_calculate_discount_zero():
    assert calculate_discount(100, 0) == 100.0

def test_calculate_discount_full():
    assert calculate_discount(100, 100) == 0.0

def test_calculate_discount_invalid():
    with pytest.raises(ValueError):
        calculate_discount(100, -5)
```

---

### Level 2: Integration Testing

Tests **how multiple components work together**.

**Examples:**
- Testing that the UserService correctly calls the UserRepository and saves to the database
- Testing that the payment API integrates with the order service

```python
def test_user_registration_flow():
    db = TestDatabase()
    email_service = MockEmailService()
    user_service = UserService(db, email_service)

    user = user_service.register("alice@example.com", "securePass123")

    assert db.find_user("alice@example.com") is not None
    assert email_service.was_called_with("alice@example.com")
```

---

### Level 3: System Testing

Tests the **entire system end-to-end** against the requirements.

Types include:
- **Functional testing** – Does it do what it should?
- **Performance testing** – Does it handle load?
- **Security testing** – Are there vulnerabilities?
- **Compatibility testing** – Does it work on Chrome, Firefox, Safari?

---

### Level 4: Acceptance Testing

Validates the system against **business requirements** and user needs.

| Type | Description |
|---|---|
| **UAT (User Acceptance Testing)** | Real users validate the system before go-live |
| **Alpha Testing** | Internal users test before external release |
| **Beta Testing** | External users test the pre-release version |

---

## 5.4 Black-Box vs. White-Box Testing

### Black-Box Testing

The tester has **no knowledge of internal code**. Tests are based only on inputs and expected outputs.

**Techniques:**

**Equivalence Partitioning:** Divide inputs into groups that should behave the same way.

For a field that accepts ages 18–65:
- Invalid partition: age < 18
- Valid partition: 18 ≤ age ≤ 65
- Invalid partition: age > 65

**Boundary Value Analysis:** Test at the edges of input ranges.

For age 18–65, test: 17, 18, 19, 64, 65, 66

**Decision Table Testing:** Test combinations of conditions.

---

### White-Box Testing (Structural Testing)

The tester **knows the internal code structure** and designs tests to cover code paths.

**Coverage metrics:**
- **Statement coverage:** Has every line of code been executed?
- **Branch coverage:** Has every if/else branch been tested?
- **Path coverage:** Has every possible execution path been tested?

```python
def get_price_tier(quantity):
    if quantity < 10:
        return "retail"
    elif quantity < 100:
        return "wholesale"
    else:
        return "bulk"

# To achieve 100% branch coverage, need tests for:
# - quantity = 5 (retail)
# - quantity = 50 (wholesale)
# - quantity = 200 (bulk)
```

---

## 5.5 Test-Driven Development (TDD) Revisited

(Introduced in Module 4, expanded here)

TDD cycle: **Red → Green → Refactor**

### Full TDD Example: Shopping Cart

```python
# --- STEP 1: Write failing tests ---
class TestShoppingCart:
    def test_empty_cart_total_is_zero(self):
        cart = ShoppingCart()
        assert cart.total() == 0

    def test_add_item_increases_total(self):
        cart = ShoppingCart()
        cart.add_item("Apple", 1.50, quantity=2)
        assert cart.total() == 3.00

    def test_remove_item(self):
        cart = ShoppingCart()
        cart.add_item("Apple", 1.50, quantity=2)
        cart.remove_item("Apple")
        assert cart.total() == 0

# --- STEP 2: Write code to pass tests ---
class ShoppingCart:
    def __init__(self):
        self.items = {}

    def add_item(self, name, price, quantity=1):
        self.items[name] = price * quantity

    def remove_item(self, name):
        if name in self.items:
            del self.items[name]

    def total(self):
        return sum(self.items.values())

# --- STEP 3: Refactor if needed ---
```

---

## 5.6 Defect Lifecycle

When a tester finds a bug, it goes through a formal lifecycle:

```
New → Assigned → In Progress → Fixed → Retest → Closed
                                    ↓ (if not fixed)
                                  Reopened
```

### Defect Report Fields

| Field | Example |
|---|---|
| **ID** | BUG-0234 |
| **Title** | Login fails with valid credentials on mobile |
| **Severity** | Critical |
| **Priority** | High |
| **Steps to Reproduce** | 1. Open app on iOS; 2. Enter valid credentials; 3. Tap Login |
| **Expected Result** | User is redirected to dashboard |
| **Actual Result** | Error: "Invalid credentials" is shown |
| **Environment** | iOS 17.2, iPhone 15, v2.3.1 |

### Severity vs. Priority

| | Severity (impact on system) | Priority (urgency to fix) |
|---|---|---|
| **High/High** | App crashes on login | Fix immediately |
| **High/Low** | Rare edge case data corruption | Fix in next release |
| **Low/High** | CEO's name is misspelled on homepage | Fix ASAP (political) |
| **Low/Low** | Minor UI misalignment | Fix when convenient |

---

## 5.7 Test Automation

Manual testing is slow and error-prone. Automation runs tests faster and more reliably.

### Popular Testing Frameworks

| Language | Unit Testing | E2E Testing |
|---|---|---|
| Python | pytest, unittest | Selenium, Playwright |
| JavaScript | Jest, Mocha | Cypress, Playwright |
| Java | JUnit, TestNG | Selenium |
| C# | NUnit, xUnit | SpecFlow |

### Continuous Integration (CI) with Automated Tests

```
Developer pushes code
        │
        ▼
  CI Server (GitHub Actions / Jenkins)
        │
        ├── Run linter (code style)
        ├── Run unit tests
        ├── Run integration tests
        ├── Check code coverage ≥ 80%
        │
        ├── PASS → ✅ Merge allowed
        └── FAIL → ❌ Developer notified, merge blocked
```

---

## 5.8 Non-Functional Testing

### Performance Testing

| Type | Goal |
|---|---|
| **Load Testing** | Can the system handle expected load? |
| **Stress Testing** | What happens beyond normal load? (find breaking point) |
| **Spike Testing** | Can the system handle sudden traffic spikes? |
| **Soak Testing** | Does performance degrade over long periods? |

**Tools:** Apache JMeter, k6, Locust, Gatling

### Security Testing

- **OWASP Top 10** – Reference for the most critical web vulnerabilities
- **Penetration Testing** – Ethically attack the system to find vulnerabilities
- **Static Analysis** – Scan source code for security issues (e.g., SonarQube)

---

## 5.9 Case Study: Testing an Online Banking App

**Scenario:** QA team for a banking app must validate a new "Transfer Money" feature.

**Test Plan:**

| Test Type | What Is Tested |
|---|---|
| Unit Tests | `calculateFee()`, `validateIBAN()`, `getAccountBalance()` functions |
| Integration Tests | Transfer Service ↔ Account Service ↔ Notification Service |
| System Tests | Full transfer flow from login to confirmation |
| UAT | 10 bank employees test real transfer scenarios |
| Performance Tests | 5,000 concurrent transfers; response time < 2s |
| Security Tests | SQL injection on account number field; session token validation |

---

## 5.10 Summary

- The later a defect is found, the more expensive it is to fix.
- Testing is organized in levels: Unit → Integration → System → Acceptance.
- Black-box testing validates behavior; white-box tests code paths.
- TDD drives quality through writing tests before code.
- Automated testing and CI ensure continuous quality in every commit.

---

## Quiz – Module 5

---

**1. According to the Testing Pyramid, which type of test should there be the most of?**

- A) End-to-End Tests
- B) Integration Tests
- C) Unit Tests ✅
- D) Acceptance Tests

---

**2. What is the key characteristic of black-box testing?**

- A) Tests every line of code
- B) Requires knowledge of internal code
- C) Tests based only on inputs and expected outputs ✅
- D) Only tests the database layer

---

**3. What does "branch coverage" measure?**

- A) Whether every function is called
- B) Whether every if/else condition has been executed ✅
- C) Whether the database is tested
- D) The number of test cases

---

**4. In the defect lifecycle, what happens after a bug is fixed?**

- A) The defect is immediately closed
- B) The bug is reassigned to a new developer
- C) The tester retests to verify the fix ✅
- D) A new bug report is filed

---

**5. Which tool is commonly used for performance/load testing?**

- A) pytest
- B) JUnit
- C) Cypress
- D) Apache JMeter ✅

---

**6. Short Answer:** Explain the difference between severity and priority in defect reporting. Give an example of a low-severity, high-priority bug.

---

**7. Practical Exercise:** Write 4 unit tests (in pseudocode or Python) for the following function:

```python
def calculate_shipping(weight_kg, is_express):
    if weight_kg <= 0:
        raise ValueError("Weight must be positive")
    base_rate = weight_kg * 2.5
    return base_rate * 1.5 if is_express else base_rate
```

---

## Further Reading

- Myers, G.J. *The Art of Software Testing* (3rd ed.)
- Beck, K. *Test-Driven Development: By Example*
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [pytest Documentation](https://docs.pytest.org)

---

*End of Module 5 | Next: Module 6 – Software Maintenance and Evolution*

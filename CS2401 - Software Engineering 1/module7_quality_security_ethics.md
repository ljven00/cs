# Module 7: Software Quality Assurance, Security, and Ethical Considerations

**Course:** CS 2401 – Software Engineering 1
**Module:** 7 of 8

---

## Learning Objectives

By the end of this module, students will be able to:
- Define software quality and its attributes
- Understand Quality Assurance vs. Quality Control
- Describe key security vulnerabilities (OWASP Top 10)
- Apply secure coding practices
- Identify ethical responsibilities of software engineers (ACM/IEEE Code of Ethics)
- Understand privacy, intellectual property, and professional accountability

---

## 7.1 What Is Software Quality?

**Software quality** is the degree to which software meets its requirements, expectations, and implicit standards.

### ISO/IEC 25010 Quality Model

The ISO/IEC 25010 standard defines 8 quality characteristics:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Software Quality (ISO 25010)                  │
├──────────────┬──────────────┬────────────┬──────────────────────┤
│ Functional   │ Performance  │ Security   │ Compatibility        │
│ Suitability  │ Efficiency   │            │                      │
├──────────────┼──────────────┼────────────┼──────────────────────┤
│ Usability    │ Reliability  │Maintainab. │ Portability          │
└──────────────┴──────────────┴────────────┴──────────────────────┘
```

| Characteristic | Meaning |
|---|---|
| **Functional Suitability** | Does it do what it's supposed to do? |
| **Performance Efficiency** | Does it perform within acceptable resource usage? |
| **Compatibility** | Can it work with other systems? |
| **Usability** | Can users effectively accomplish their goals? |
| **Reliability** | Does it perform consistently under specified conditions? |
| **Security** | Does it protect data and resources from unauthorized access? |
| **Maintainability** | Can it be modified and updated efficiently? |
| **Portability** | Can it be transferred to different environments? |

---

## 7.2 Quality Assurance vs. Quality Control

These are often confused but are fundamentally different:

| | Quality Assurance (QA) | Quality Control (QC) |
|---|---|---|
| **Focus** | Processes | Product |
| **Goal** | Prevent defects | Detect defects |
| **When** | Throughout development | After development |
| **Activity** | Code reviews, process audits, standards | Testing, inspection |
| **Approach** | Proactive | Reactive |

**Analogy:** QA is like training chefs to cook properly (preventing bad food). QC is like a food taster checking the dish before it's served (detecting bad food).

---

## 7.3 Quality Metrics

| Metric | Formula / Description |
|---|---|
| **Defect Density** | Defects per 1,000 lines of code (KLOC) |
| **Mean Time to Failure (MTTF)** | Average time between failures in production |
| **Mean Time to Repair (MTTR)** | Average time to fix a failure |
| **Test Coverage** | % of code exercised by automated tests |
| **Code Churn** | % of code changed in a given period |
| **Customer-Reported Defects** | Number of bugs found by end users |

---

## 7.4 Software Security

Security is not an add-on — it must be built in from the start.

> "Security is not a product, but a process." — Bruce Schneier

### Secure Development Lifecycle (SDL)

```
Training → Requirements → Design → Implementation → Verification → Release → Response
             (security     (threat    (secure      (security       (patch
              req'ts)      model)    coding)       testing)        mgmt)
```

---

### OWASP Top 10 Web Application Security Risks

The **Open Web Application Security Project (OWASP)** publishes the most critical web security vulnerabilities.

#### 1. Broken Access Control
Users can access resources they are not authorized for.

```python
# VULNERABLE: No authorization check
@app.route("/admin/delete_user/<user_id>")
def delete_user(user_id):
    db.delete_user(user_id)  # Any user can call this!

# SECURE: Check role
@app.route("/admin/delete_user/<user_id>")
@require_role("admin")
def delete_user(user_id):
    db.delete_user(user_id)
```

#### 2. Cryptographic Failures
Sensitive data exposed due to weak or missing encryption.

```python
# VULNERABLE: Storing plain text passwords
user.password = "mypassword123"

# SECURE: Hashing with bcrypt
import bcrypt
hashed = bcrypt.hashpw("mypassword123".encode(), bcrypt.gensalt())
user.password_hash = hashed
```

#### 3. Injection (SQL Injection)
Malicious input is interpreted as code.

```python
# VULNERABLE: String concatenation in SQL
query = "SELECT * FROM users WHERE email = '" + user_input + "'"
# Input: ' OR '1'='1  → returns ALL users!

# SECURE: Parameterized queries
cursor.execute("SELECT * FROM users WHERE email = %s", (user_input,))
```

#### 4. Insecure Design
Fundamental design flaws that cannot be fixed by better coding alone.

**Example:** A password reset system that resets to a predictable default password.

#### 5. Security Misconfiguration
Default passwords, unnecessary services, verbose error messages.

```python
# VULNERABLE: Verbose error in production
except Exception as e:
    return {"error": str(e), "traceback": traceback.format_exc()}

# SECURE: Generic error to user, detailed log server-side
except Exception as e:
    logger.error(f"Internal error: {e}", exc_info=True)
    return {"error": "An internal error occurred"}, 500
```

#### 6. Vulnerable and Outdated Components
Using libraries with known vulnerabilities.

**Fix:** Regularly run `npm audit`, `pip-audit`, or use Dependabot/Snyk.

#### 7. Identification and Authentication Failures
Weak session management, no multi-factor authentication.

#### 8. Software and Data Integrity Failures
Code or data updated without integrity verification (e.g., unverified software updates).

#### 9. Security Logging and Monitoring Failures
No logging of security events → attacks go undetected for months.

#### 10. Server-Side Request Forgery (SSRF)
Attacker tricks the server into making requests to internal resources.

---

## 7.5 Secure Coding Principles

| Principle | Description |
|---|---|
| **Least Privilege** | Grant the minimum access necessary |
| **Defense in Depth** | Multiple layers of security controls |
| **Fail Securely** | On error, default to a safe state |
| **Validate All Inputs** | Never trust data from outside the system |
| **Don't Roll Your Own Crypto** | Use proven, standard cryptographic libraries |
| **Keep Secrets Out of Code** | Use environment variables, not hardcoded credentials |

```python
# BAD: Secret hardcoded in source code
API_KEY = "sk-live-abc123xyz"

# GOOD: Read from environment
import os
API_KEY = os.environ.get("API_KEY")
```

---

## 7.6 Ethics in Software Engineering

Software engineers have a unique responsibility. The systems we build affect millions of people's lives, privacy, safety, and freedoms.

### ACM/IEEE Code of Ethics (8 Principles)

The ACM-IEEE Software Engineering Code of Ethics requires software engineers to act consistently with the public interest in 8 principle areas:

| Principle | Core Idea |
|---|---|
| **1. Public** | Act in the public interest |
| **2. Client and Employer** | Act in the best interest of clients, but in a manner consistent with public interest |
| **3. Product** | Ensure products meet the highest professional standards |
| **4. Judgment** | Maintain integrity and independence in professional judgment |
| **5. Management** | Promote ethical approaches in management of software development |
| **6. Profession** | Advance the integrity and reputation of the profession |
| **7. Colleagues** | Be fair to and supportive of colleagues |
| **8. Self** | Participate in lifelong learning and promote ethical practice |

---

### Real-World Ethical Dilemmas

#### Case 1: Volkswagen Emissions Scandal (2015)
VW engineers wrote software that detected when a car was being emissions-tested and switched to a cleaner mode. Under real driving conditions, the car emitted up to 40× the legal NOx limit.

**Ethical violations:** Deception of customers and regulators; public health harm; professional dishonesty.

---

#### Case 2: Facebook and Cambridge Analytica (2018)
Data from 87 million Facebook profiles was harvested without consent and used for political targeting.

**Ethical issues:** Privacy violations; misuse of user data; failure to prevent third-party abuse.

---

#### Case 3: Self-Driving Car Accidents
Who is responsible when an autonomous vehicle causes a fatal accident — the developer, the company, or the user?

**Ethical considerations:** Transparency of decision-making algorithms; legal liability; informed consent.

---

### Privacy and Data Protection

**Key principles (GDPR-aligned):**
- **Lawful basis:** Data must only be collected with a legal reason
- **Purpose limitation:** Data collected for one purpose cannot be used for another
- **Data minimization:** Collect only what is necessary
- **Storage limitation:** Do not keep data longer than needed
- **User rights:** Right to access, correct, and delete their data

---

### Intellectual Property

| Concept | Description |
|---|---|
| **Copyright** | Automatic protection of original creative works (code, documentation) |
| **Patent** | Exclusive rights to an invention for 20 years |
| **Trademark** | Brand identifiers (logos, names) |
| **Trade Secret** | Confidential business information (algorithms, recipes) |
| **Open Source Licenses** | GPL, MIT, Apache — grant specific usage rights with conditions |

**Practical implication:** Using GPL-licensed code in a commercial product may require you to open-source your product too (copyleft).

---

### Whistleblowing

When a software engineer discovers serious ethical violations:
1. Address internally first (to management, ethics board)
2. If unresolved and harm is serious, external disclosure may be justified
3. Legal protections for whistleblowers vary by country

---

## 7.7 Accessibility and Inclusive Design

Software must be usable by people of all abilities.

**WCAG 2.1 (Web Content Accessibility Guidelines)** defines 4 principles:
- **Perceivable** – Content can be perceived by all senses
- **Operable** – UI is operable by keyboard and assistive tech
- **Understandable** – Content is readable and predictable
- **Robust** – Works across browsers and assistive technologies

**Practical checklist:**
- Provide alt text for images
- Ensure sufficient color contrast (4.5:1 for normal text)
- Support keyboard navigation
- Provide captions for video content
- Design for screen readers (proper ARIA labels)

---

## 7.8 Summary

- Software quality encompasses functional suitability, performance, security, usability, and more.
- QA is process-focused (prevention); QC is product-focused (detection).
- The OWASP Top 10 covers the most critical web security vulnerabilities.
- Secure coding: validate inputs, use parameterized queries, apply least privilege, keep secrets out of code.
- The ACM/IEEE Code of Ethics defines 8 professional obligations.
- Software engineers must consider privacy, IP, accessibility, and social responsibility.

---

## Quiz – Module 7

---

**1. Which ISO 25010 quality characteristic asks "does it protect against unauthorized access?"**

- A) Reliability
- B) Usability
- C) Security ✅
- D) Portability

---

**2. SQL Injection is best prevented by:**

- A) Encrypting the database
- B) Using parameterized queries ✅
- C) Limiting the number of login attempts
- D) Hashing user passwords

---

**3. What is the primary difference between QA and QC?**

- A) QA detects defects; QC prevents them
- B) QA is done after development; QC during
- C) QA focuses on processes; QC focuses on the product ✅
- D) QA is automated; QC is manual

---

**4. According to the ACM/IEEE Code of Ethics, which principle comes first?**

- A) Employer
- B) Self
- C) Product
- D) Public ✅

---

**5. Storing passwords in plain text violates which OWASP Top 10 category?**

- A) Injection
- B) Broken Access Control
- C) Cryptographic Failures ✅
- D) Insecure Design

---

**6. Short Answer:** A developer discovers that their company's app is quietly collecting location data from users without their knowledge or consent. What ethical obligations does the developer have? What steps should they take?

---

**7. Practical Exercise:** Review the following code snippet and identify ALL security issues:

```python
@app.route("/user")
def get_user():
    user_id = request.args.get("id")
    query = f"SELECT * FROM users WHERE id = {user_id}"
    result = db.execute(query)
    return jsonify(result)
```

---

## Further Reading

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [ACM Code of Ethics](https://www.acm.org/code-of-ethics)
- [WCAG 2.1 Guidelines](https://www.w3.org/TR/WCAG21/)
- Anderson, R.J. *Security Engineering* (3rd ed.)
- Floridi, L. *The Ethics of Artificial Intelligence*

---

*End of Module 7 | Next: Module 8 – Emerging Trends in Software Engineering*

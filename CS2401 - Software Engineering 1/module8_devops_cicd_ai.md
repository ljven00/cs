# Module 8: Emerging Trends in Software Engineering

**Course:** CS 2401 – Software Engineering 1
**Module:** 8 of 8

---

## Learning Objectives

By the end of this module, students will be able to:
- Explain the principles and practices of DevOps
- Describe the CI/CD pipeline and its components
- Understand Infrastructure as Code (IaC)
- Identify ways AI and ML are transforming software engineering
- Reflect on the future of the profession

---

## 8.1 DevOps: Bridging Development and Operations

### What is DevOps?

**DevOps** is a cultural and technical movement that unifies software development (Dev) and IT operations (Ops). Its goal is to shorten the development lifecycle and deliver high-quality software continuously.

> "DevOps is not a tool, not a title, and not a team. It's a culture." — Gene Kim

### The Three Ways of DevOps (Gene Kim)

```
First Way:    Flow         Dev → Ops → Customer (fast delivery)
Second Way:   Feedback     Customer → Ops → Dev (fast feedback)
Third Way:    Continuous   Culture of experimentation and learning
              Learning
```

---

### DevOps vs. Traditional IT

| | Traditional | DevOps |
|---|---|---|
| Release cadence | Months/years | Days/weeks |
| Dev ↔ Ops relationship | Siloed, blame culture | Collaborative |
| Deployments | Manual, risky | Automated, frequent |
| Failure response | Blame, long recovery | Learn, fast recovery |
| Infrastructure | Manual provisioning | Infrastructure as Code |

---

### DORA Metrics (DevOps Performance)

The DevOps Research and Assessment (DORA) team identified 4 key metrics:

| Metric | Elite Teams Benchmark |
|---|---|
| **Deployment Frequency** | Multiple times per day |
| **Lead Time for Changes** | Less than 1 hour |
| **Mean Time to Restore (MTTR)** | Less than 1 hour |
| **Change Failure Rate** | 0–15% |

---

## 8.2 Continuous Integration (CI)

**CI** is the practice of developers integrating code into a shared repository **multiple times per day**, with each integration verified by automated builds and tests.

### CI Pipeline

```
Developer pushes code
         │
         ▼
  Source Control (Git)
         │
         ▼
  CI Server triggered (GitHub Actions / Jenkins / GitLab CI)
         │
         ├─── 1. Checkout code
         ├─── 2. Install dependencies
         ├─── 3. Run linter (code style check)
         ├─── 4. Run unit tests
         ├─── 5. Run integration tests
         ├─── 6. Build artifact (Docker image / JAR / ZIP)
         ├─── 7. Security scan (SAST)
         └─── 8. Code coverage check
                │
         ┌──────┴──────┐
        PASS          FAIL
         │              │
    Merge allowed   Notify developer
```

### GitHub Actions Example

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run linter
        run: flake8 src/
      
      - name: Run tests
        run: pytest --cov=src tests/
      
      - name: Check coverage
        run: coverage report --fail-under=80
```

---

## 8.3 Continuous Delivery vs. Continuous Deployment

```
                 CI            CD (Delivery)       CD (Deployment)
                 │                  │                     │
Code ──► Build ──► Test ──► Deploy to Staging ──► [Gate] ──► Production
                                                    │              │
                                           Manual approval    Automatic
                                           (Delivery)         (Deployment)
```

| | Continuous Delivery | Continuous Deployment |
|---|---|---|
| To staging | Automatic | Automatic |
| To production | Manual approval required | Fully automatic |
| Risk | Lower (human gate) | Higher (fully automated) |
| Speed | Fast | Fastest |

**Example:** Netflix deploys to production **thousands of times per day** using continuous deployment.

---

## 8.4 Infrastructure as Code (IaC)

**IaC** means managing and provisioning infrastructure through **code rather than manual processes**.

### Why IaC?

```
Manual Infrastructure         Infrastructure as Code
──────────────────────────    ─────────────────────────────
Click through AWS console     Write code to define resources
Undocumented configuration    Version-controlled in Git
Hard to reproduce             Reproducible anywhere
Inconsistent environments     Identical dev/staging/prod
```

### Terraform Example

```hcl
# main.tf — provision an AWS web server
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name        = "WebServer"
    Environment = "Production"
  }
}

resource "aws_security_group" "web_sg" {
  name = "web-security-group"

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Popular IaC Tools

| Tool | Provider | Focus |
|---|---|---|
| **Terraform** | HashiCorp | Multi-cloud provisioning |
| **AWS CloudFormation** | Amazon | AWS-specific |
| **Ansible** | Red Hat | Configuration management |
| **Pulumi** | Pulumi Corp | IaC with real programming languages |
| **Kubernetes (K8s)** | CNCF | Container orchestration |

---

## 8.5 Containers and Docker

**Containers** package an application with all its dependencies into an isolated, portable unit.

```
WITHOUT CONTAINERS            WITH CONTAINERS
─────────────────────         ──────────────────────────────
"Works on my machine!"         Runs identically everywhere

Developer laptop (works)       ┌────────────────────────────┐
Staging server (fails –        │       Docker Container     │
  different Python version)    │  App + Dependencies + OS   │
Production (crashes –          │  libs + Config             │
  different env vars)          └────────────────────────────┘
                                Works on any machine with Docker
```

### Dockerfile Example

```dockerfile
# Dockerfile for a Python Flask app
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

### Docker Compose (multi-service)

```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql://postgres:secret@db:5432/myapp

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
```

---

## 8.6 AI in Software Engineering

AI is transforming how software is built, at every stage of the SDLC.

### AI Applications Across the SDLC

| SDLC Phase | AI Application |
|---|---|
| **Requirements** | NLP to extract requirements from documents; auto-generate user stories |
| **Design** | AI-suggested architecture patterns based on project type |
| **Development** | Code completion (GitHub Copilot), code generation (ChatGPT, Claude) |
| **Testing** | Auto-generating test cases, AI-driven test exploration |
| **Code Review** | Detecting security vulnerabilities (CodeQL), suggesting improvements |
| **Deployment** | Predictive rollback, anomaly detection in logs |
| **Maintenance** | Automatically classifying and triaging bug reports |

---

### AI-Powered Developer Tools (2025)

| Tool | Capability |
|---|---|
| **GitHub Copilot** | In-editor code completion and generation |
| **Cursor** | AI-first code editor |
| **Claude (Anthropic)** | Code review, refactoring, documentation |
| **CodeWhisperer** | Amazon's AI code assistant |
| **Tabnine** | Privacy-focused AI code completion |
| **SonarQube + AI** | AI-enhanced static code analysis |

---

### Responsible AI in Software Engineering

AI code generation introduces new risks:

| Risk | Mitigation |
|---|---|
| AI generates insecure code | Always review AI output; run security scans |
| AI introduces biased logic | Audit algorithms for fairness |
| Over-reliance on AI | Maintain foundational coding skills |
| Copyright infringement | Check licensing of AI-generated code |
| Hallucinated APIs | Test all generated code, don't trust blindly |

> "AI is a powerful assistant, not a replacement for engineering judgment."

---

## 8.7 Site Reliability Engineering (SRE)

**SRE** (pioneered by Google) applies software engineering principles to operations problems.

### Key SRE Concepts

| Concept | Description |
|---|---|
| **SLI** (Service Level Indicator) | A metric that measures service behavior (e.g., latency, error rate) |
| **SLO** (Service Level Objective) | Target value for an SLI (e.g., 99.9% availability) |
| **SLA** (Service Level Agreement) | Contract with customers defining consequences for missing SLOs |
| **Error Budget** | Allowed downtime/errors before SLO is violated |
| **Toil** | Manual, repetitive operational work that should be automated |

**Example:** If the SLO is 99.9% uptime, the error budget is 8.7 hours/year of allowable downtime.

---

## 8.8 Cloud-Native Development

**Cloud-native** means designing applications specifically for cloud environments, taking advantage of elasticity, managed services, and global distribution.

### Cloud-Native Principles (12-Factor App)

Key factors include:
1. **Codebase:** One codebase tracked in version control
2. **Dependencies:** Explicitly declared and isolated
3. **Config:** Stored in environment, not code
4. **Backing services:** Treat databases and APIs as attached resources
5. **Build/Release/Run:** Strictly separated stages
6. **Processes:** Stateless, share-nothing processes
7. **Port binding:** Export services via port binding
8. **Concurrency:** Scale out via the process model
9. **Disposability:** Fast startup, graceful shutdown
10. **Dev/Prod parity:** Keep environments as similar as possible
11. **Logs:** Treat logs as event streams
12. **Admin processes:** Run admin tasks as one-off processes

---

## 8.9 The Future of Software Engineering

### Trends Shaping the Profession

1. **Low-code / No-code platforms** – Democratizing software creation for non-developers; software engineers focus on platforms and infrastructure
2. **AI-augmented development** – Engineers direct AI rather than write every line
3. **Quantum computing** – Future impact on cryptography and optimization
4. **Edge computing** – Processing closer to the device (IoT, autonomous vehicles)
5. **Green software engineering** – Measuring and reducing carbon footprint of software
6. **Platform engineering** – Internal developer platforms that abstract infrastructure

### Skills for the Future Software Engineer

```
Technical                    Soft Skills
─────────────────────────    ────────────────────────
Cloud platforms (AWS/GCP)    Systems thinking
DevOps tooling               Communication across teams
Security-first mindset       Continuous learning
AI tool proficiency          Ethical reasoning
Data engineering             Adaptability to new paradigms
Observability & monitoring   Collaboration in distributed teams
```

---

## 8.10 Course Reflection and Synthesis

You have now covered the full spectrum of Software Engineering 1:

| Module | Topic |
|---|---|
| 1 | Introduction to SE and SDLC |
| 2 | Requirements Engineering |
| 3 | Software Design and Architecture |
| 4 | Development Methodologies |
| 5 | Testing and QA |
| 6 | Maintenance and Evolution |
| 7 | Quality, Security, and Ethics |
| 8 | DevOps, CI/CD, and AI in SE |

These topics are not isolated — they form an **interconnected system:**
- Good requirements prevent defects
- Good design makes testing and maintenance easier
- Agile methodologies keep the team aligned with user needs
- CI/CD delivers quality software fast
- Ethics and security ensure software is trustworthy

---

## 8.11 Summary

- DevOps unifies development and operations, enabling faster and more reliable delivery.
- CI/CD pipelines automate build, test, and deployment, catching issues early.
- Infrastructure as Code makes environments reproducible and version-controlled.
- Containers (Docker) ensure consistency across all environments.
- AI is transforming every phase of the SDLC but requires engineering judgment.
- Cloud-native and SRE practices define modern software operations.

---

## Quiz – Module 8

---

**1. Which DORA metric measures how quickly a team can recover from a production failure?**

- A) Deployment Frequency
- B) Lead Time for Changes
- C) Change Failure Rate
- D) Mean Time to Restore (MTTR) ✅

---

**2. What is the difference between Continuous Delivery and Continuous Deployment?**

- A) Delivery tests automatically; Deployment tests manually
- B) Delivery requires manual approval to production; Deployment is fully automatic ✅
- C) Deployment is only for staging; Delivery is for production
- D) There is no difference

---

**3. What is Infrastructure as Code (IaC)?**

- A) Writing application code that runs on servers
- B) Using code to provision and manage infrastructure instead of manual processes ✅
- C) Deploying containers to cloud platforms
- D) Writing documentation for infrastructure

---

**4. In SRE, what is an "error budget"?**

- A) The financial cost of system errors
- B) The number of bugs allowed per sprint
- C) The allowable amount of downtime/errors before an SLO is violated ✅
- D) A Jira label for critical bugs

---

**5. Which of the following is a risk of using AI-generated code?**

- A) Code runs too fast
- B) AI always follows current best practices
- C) AI may generate insecure or incorrect code that developers blindly accept ✅
- D) AI cannot generate code in Python

---

**6. Short Answer:** Explain the Three Ways of DevOps. How do they apply to a software team you've been part of or can imagine working on?

---

**7. Final Synthesis Exercise:** You are the software engineering lead for a 10-person startup launching a fintech app. Describe:
- Which SDLC methodology you'd choose and why
- How you'd set up your CI/CD pipeline
- What DevOps practices you'd implement from day one
- How you'd address security and ethical considerations
- How you'd use AI tools responsibly in your workflow

---

## Course Completion Checklist

Before your final exam, make sure you can:
- [ ] Describe all 7 SDLC phases
- [ ] Write functional and non-functional requirements
- [ ] Create a use case diagram and a class diagram
- [ ] Explain Scrum roles, events, and artifacts
- [ ] Write and explain unit tests
- [ ] Identify the 4 types of maintenance
- [ ] Name and explain 5 OWASP Top 10 vulnerabilities
- [ ] Explain the ACM/IEEE Code of Ethics principles
- [ ] Describe a full CI/CD pipeline
- [ ] Explain how AI is impacting software engineering

---

## Further Reading

- Kim, G. et al. *The DevOps Handbook*
- Kim, G. *The Phoenix Project*
- Burns, B. et al. *Kubernetes: Up and Running* (3rd ed.)
- [12 Factor App](https://12factor.net)
- [Google SRE Book](https://sre.google/sre-book/table-of-contents/) (free online)
- [GitHub Copilot Documentation](https://docs.github.com/en/copilot)

---

*End of Module 8 — Course Complete!*

---

**CS 2401 Software Engineering 1 — All Modules**

| # | Module | File |
|---|---|---|
| 1 | Introduction to SE and SDLC | module1_intro_to_se_and_sdlc.md |
| 2 | Requirements Engineering | module2_requirements_engineering.md |
| 3 | Software Design and Architecture | module3_software_design_architecture.md |
| 4 | Development Methodologies | module4_development_methodologies.md |
| 5 | Testing and QA | module5_testing_and_qa.md |
| 6 | Maintenance and Evolution | module6_maintenance_and_evolution.md |
| 7 | Quality, Security, and Ethics | module7_quality_security_ethics.md |
| 8 | DevOps, CI/CD, and AI in SE | module8_devops_cicd_ai.md |

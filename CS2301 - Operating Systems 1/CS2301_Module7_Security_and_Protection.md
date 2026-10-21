# CS 2301 — Operating Systems 1
## Module 7: Security and Protection

---

> **Course Progress:** Module 7 of 8 — CS 2301 (OS 1)

---

## 1. Security vs. Protection

These two terms are related but distinct:

| Term | Meaning |
|------|---------|
| **Protection** | Mechanisms that control access to resources within the system (who can read which file, which process can use which memory) |
| **Security** | Defending the system against external and internal threats — unauthorized users, malicious software, attacks |

Protection is a tool; security is the goal. The OS provides both.

---

## 2. Goals of OS Security

A secure OS aims to guarantee three properties, often called the **CIA Triad**:

```
        Confidentiality
             /\
            /  \
           /    \
          /      \
    Integrity ── Availability
```

- **Confidentiality** — data is only accessible to authorized parties
- **Integrity** — data is only modified by authorized parties in authorized ways
- **Availability** — authorized users can access the system when needed

A successful attack violates at least one of these.

---

## 3. Authentication

**Authentication** is how the OS verifies who you are before granting access.

### Three Factors of Authentication

| Factor | What it is | Example |
|--------|-----------|---------|
| **Something you know** | Secret knowledge | Password, PIN |
| **Something you have** | Physical object | Smart card, phone (OTP) |
| **Something you are** | Biometric | Fingerprint, face scan |

**Multi-factor authentication (MFA)** requires two or more factors — much harder to compromise than a password alone.

### Password Storage

Storing passwords in plain text is a critical mistake. The OS stores passwords as **hashes** — one-way transformations.

```
Password: "hunter2"
   ↓  hash function (e.g., bcrypt)
Stored:  "$2b$12$eImiTXuWVxfM37uY4JANj..."
```

When you log in, the OS hashes what you typed and compares it to the stored hash. The original password is never stored.

**Salt** — a random value added to the password before hashing — ensures two users with the same password have different stored hashes, defeating pre-computed attack tables.

---

## 4. Access Control

Once authenticated, the OS must decide what an authenticated user or process is **authorized** to do.

### 4.1 Access Control Matrix

A conceptual table mapping every subject (user/process) to every object (file/device) with allowed operations.

```
         File A    File B    Printer
Alice:   read/write  read     print
Bob:     read        ---      ---
Root:    read/write  read/write  print
```

In practice, storing this full matrix is impractical — most entries are empty.

### 4.2 Access Control Lists (ACL)

Store the matrix **per object** — each file/resource has a list of who can do what.

```
File A:
  - Alice: read, write
  - Bob:   read
  - Root:  read, write, execute
```

Unix uses a simplified ACL: owner / group / others, each with read (r), write (w), execute (x).

```bash
$ ls -l file.txt
-rw-r--r--  1 alice staff  1024 May 1 10:00 file.txt
 ↑↑↑ ↑↑↑ ↑↑↑
 │   │   └── others: read only
 │   └────── group:  read only
 └────────── owner:  read + write
```

### 4.3 Capability Lists

Store the matrix **per subject** — each user/process carries a list of what they are allowed to do (like a set of keys).

### 4.4 Role-Based Access Control (RBAC)

Instead of assigning permissions to individual users, assign them to **roles** (e.g., Admin, Editor, Viewer), then assign users to roles.

```
User Alice → Role: Editor → Permissions: read, write files
User Bob   → Role: Viewer → Permissions: read files only
```

RBAC is widely used in enterprise systems and cloud platforms.

---

## 5. The Principle of Least Privilege

> **Give every user, process, and system component only the minimum access needed to do its job — nothing more.**

This limits the damage if something goes wrong:
- A compromised browser can't delete system files if it doesn't have permission
- A bug in a web server can't read your private keys if it doesn't have access
- A regular user account can't overwrite OS files

This is why Linux and macOS ask for a password before installing software, and why mobile apps request specific permissions.

---

## 6. Common Threats and Attacks

### 6.1 Malware

Malicious software designed to damage, disrupt, or gain unauthorized access.

| Type | Description |
|------|-------------|
| **Virus** | Attaches to legitimate programs; spreads when they run |
| **Worm** | Self-replicating; spreads across networks without user action |
| **Trojan** | Disguised as legitimate software; hides malicious payload |
| **Ransomware** | Encrypts your files; demands payment for the key |
| **Spyware** | Silently collects data (keystrokes, passwords, browsing) |
| **Rootkit** | Hides itself deep in the OS; gives attacker persistent access |

### 6.2 Buffer Overflow

A program writes more data into a buffer than it can hold, overwriting adjacent memory — including the **return address** on the stack. An attacker can redirect execution to their own code.

```c
void vulnerable(char *input) {
    char buffer[64];
    strcpy(buffer, input);  // No bounds check!
}
// If input is 200 bytes, it overflows the stack
```

This is one of the most exploited vulnerabilities in history. Defenses include:
- **Stack canaries** — a sentinel value placed on the stack; if overwritten, the OS kills the process
- **Address Space Layout Randomization (ASLR)** — randomizes memory layout so attackers can't predict where their code lands
- **Non-executable stack (NX/DEP)** — marks stack memory as non-executable

### 6.3 Privilege Escalation

An attacker starts with limited access and exploits a vulnerability to gain higher privileges (e.g., from regular user to root/admin).

### 6.4 Social Engineering

Tricking a human into compromising security — phishing emails, fake login pages, impersonating IT support. The OS can't protect against this; it's a human problem.

### 6.5 Denial of Service (DoS)

Overwhelming a system with requests until it can no longer serve legitimate users. Targets **availability**.

---

## 7. OS Protection Mechanisms

The OS itself has several built-in mechanisms to enforce protection:

### 7.1 Dual-Mode Operation

The CPU runs in either **user mode** or **kernel mode** (covered in Module 1). User processes cannot directly access hardware or other processes' memory — the OS enforces this at the hardware level.

### 7.2 Memory Protection

Each process has its own address space. The MMU (Module 4) enforces boundaries — a process trying to access another's memory triggers a **segmentation fault** and is killed.

### 7.3 Timer Interrupts

The OS sets a hardware timer. If a process runs too long without yielding, the timer fires an interrupt and the OS regains control. This prevents a process from monopolizing the CPU forever.

### 7.4 File Permissions

The file system enforces read/write/execute permissions per user and group (as seen in section 4.2).

### 7.5 System Call Validation

Every system call is validated by the kernel — pointers are checked, permissions are verified. A process can't trick the kernel into doing something unauthorized through a system call.

---

## 8. Encryption

**Encryption** transforms data so only authorized parties can read it, even if an attacker gets physical access.

### Symmetric Encryption
Same key encrypts and decrypts. Fast. Problem: how do you share the key securely?

```
Plaintext + Key → [Encrypt] → Ciphertext
Ciphertext + Key → [Decrypt] → Plaintext
```

Example: AES (Advanced Encryption Standard) — used for disk encryption (BitLocker, FileVault).

### Asymmetric Encryption
A **public key** encrypts; a **private key** decrypts. You can share the public key freely.

```
Plaintext + Public Key  → [Encrypt] → Ciphertext
Ciphertext + Private Key → [Decrypt] → Plaintext
```

Example: RSA — used for HTTPS, SSH.

### Full Disk Encryption
The entire disk is encrypted. Without the correct password/key at boot, the data is unreadable — even if someone removes the drive.

---

## 9. Auditing and Logging

The OS records **audit logs** — a trail of security-relevant events:

- Login attempts (successful and failed)
- File access and modifications
- Privilege escalations
- System calls from privileged processes

Logs are critical for:
- Detecting attacks in progress
- Investigating breaches after the fact
- Compliance (regulations like HIPAA, GDPR require audit trails)

```bash
# View authentication logs on Linux
cat /var/log/auth.log

# View recent login attempts
last
lastb   # failed attempts
```

---

## 10. Real-World Example: What Happens When You Log Into Linux

1. You type your username → OS looks up the account in `/etc/passwd`
2. You type your password → OS hashes it and compares to the stored hash in `/etc/shadow`
3. Hash matches → authentication succeeds
4. OS creates a **session** for your user ID (UID)
5. Every file access, process launch, and system call is checked against your UID and permissions
6. If you run `sudo` → OS checks if your account is in the sudoers list, prompts for password again
7. Elevated process runs with root UID — **least privilege** means this window is kept as short as possible
8. All of these events are written to `/var/log/auth.log`

---

## 11. Summary

| Concept | Key Takeaway |
|---------|-------------|
| CIA Triad | Confidentiality, Integrity, Availability — the three security goals |
| Authentication | Verifying identity (know, have, are) |
| Password hashing | Passwords stored as salted hashes, never plain text |
| ACL | Per-resource list of who can do what |
| RBAC | Assign permissions to roles, users to roles |
| Least privilege | Give only the minimum access needed |
| Buffer overflow | Overwriting stack memory to hijack execution |
| ASLR / NX | OS defenses against memory attacks |
| Dual-mode operation | Hardware-enforced user vs. kernel mode |
| Encryption | AES for storage, RSA for key exchange |
| Audit logs | Record of security events for detection and investigation |

---

## Module 7 Quiz

---

**Question 1**
What is the CIA Triad and why is it important?

> **Answer:** CIA stands for Confidentiality (only authorized parties access data), Integrity (only authorized parties modify data in authorized ways), and Availability (authorized users can access the system when needed). It provides a framework for evaluating security — every attack violates at least one of these three properties.

---

**Question 2**
Why are passwords stored as hashes rather than plain text? What does "salting" add?

> **Answer:** Storing plain text passwords means anyone who reads the database (an attacker, a rogue employee) gets everyone's passwords immediately. Hashing is one-way — even if the hash is stolen, recovering the original password is computationally hard. Salting adds a unique random value to each password before hashing, ensuring two users with the same password have different hashes — this defeats pre-computed rainbow table attacks.

---

**Question 3**
A company has 500 employees. Some are admins, some are editors, some are viewers. Which access control model is most practical to manage and why?

> **Answer:** **Role-Based Access Control (RBAC).** Instead of assigning permissions to each of 500 users individually, you define three roles (Admin, Editor, Viewer) with their permissions, then assign each user to a role. Adding a new employee means assigning them a role — one step. Changing editor permissions means updating one role, not 200 individual users.

---

**Question 4**
What is a buffer overflow attack and name two OS-level defenses against it?

> **Answer:** A buffer overflow occurs when a program writes more data into a fixed-size buffer than it can hold, overwriting adjacent memory including the stack's return address. An attacker crafts input that redirects execution to malicious code. Two OS defenses are: (1) **ASLR** (Address Space Layout Randomization) — randomizes memory layout so attackers can't predict where their code will be; (2) **Non-executable stack (NX/DEP)** — marks stack memory as non-executable so injected code can't run even if the return address is hijacked.

---

**Question 5**
Look at this Unix file permission output. Who can do what with this file?

```
-rwxr-x---  1 bob developers  2048 May 1 09:00 script.sh
```

> **Answer:**
> - **Owner (bob):** read, write, execute (`rwx`)
> - **Group (developers):** read and execute only (`r-x`) — cannot write
> - **Others (everyone else):** no permissions at all (`---`) — cannot read, write, or execute
>
> So only bob can modify the script. Members of the developers group can read and run it. Everyone else is locked out entirely.

---

## What's Next

**Module 8: Advanced Topics and Case Studies**

The final module of CS 2301 — you'll bring everything together by looking at virtualization, containers, real OS case studies (Linux, Windows), and a preview of what's ahead in OS 2.

---

*CS 2301 Operating Systems 1 — Module 7 of 8*

# PASTA Threat Model — Sneaker Company Mobile App

## Project Description

In this project, I conducted a threat model of a new e-commerce mobile application using the PASTA (Process for Attack Simulation and Threat Analysis) framework. Threat modeling is a critical part of secure software development, allowing security teams to identify vulnerabilities before malicious actors can exploit them. Working through all seven stages of the PASTA framework, I defined business and security objectives, evaluated the application's technical components, analyzed threats and vulnerabilities using an attacker mindset, and recommended security controls to reduce the risk profile of the new sneaker marketplace app before its launch.

---

## Scenario

A company for sneaker enthusiasts and collectors is preparing to launch a mobile app that allows customers to buy and sell shoes. As part of the security team, I performed a threat model of the application using the PASTA framework to identify security requirements and ensure the app is safe to launch. The app handles sensitive user data including account credentials, personal information, and payment details.

---

## PASTA Worksheet

| Stages | Sneaker Company Application |
|--------|---------------------------|
| **I. Define business and security objectives** | **1.** The app must securely process financial transactions and provide users with multiple payment options for a smooth checkout, while ensuring proper payment handling to avoid legal and regulatory issues (e.g., PCI DSS compliance). **2.** The app must protect user data privacy and handle personal information responsibly, giving users confidence that their data is secure. **3.** The app must provide secure account management, allowing users to easily and safely sign up, log in, and manage their accounts, and enable buyers to message and rate sellers. |
| **II. Define the technical scope** | I would prioritize evaluating **SQL** first because the application relies on it to store and retrieve sensitive user data, seller information, and transaction records — making the database a high-value target. A poorly secured SQL implementation is vulnerable to SQL injection attacks, one of the most common and damaging web application threats, which could expose the entire database of user and payment information. Securing SQL protects the core data the entire app depends on. |
| **III. Decompose application** | Reviewed the data flow diagram. The product search process shows the User sending a search request ("Searching for sneakers for sale") to the Database, which returns "Listings of current inventory." This process must be secured to prevent injection attacks and unauthorized data access during database queries. |
| **IV. Threat analysis** | **1. External threat — Malicious hackers:** Threat actors attempting to steal user data, payment information, or account credentials through attacks such as SQL injection, session hijacking, or credential theft. **2. Internal threat — Social engineering of employees:** A threat actor could manipulate an employee into revealing credentials or access, compromising the authentication system and gaining unauthorized access to user data. |
| **V. Vulnerability analysis** | **1. Lack of prepared statements (SQL injection):** If the application does not use prepared statements or parameterized queries, attackers can inject malicious SQL code to access, alter, or delete database contents. **2. Weak login credentials:** If the app allows weak passwords or lacks account protection, attackers can use brute force or credential stuffing to gain unauthorized access to user accounts, enabling session hijacking. |
| **VI. Attack modeling** | Reviewed the attack tree. The root goal is compromising **User data**, achieved through two attack paths: **SQL injection** (enabled by lack of prepared statements) and **Session hijacking** (enabled by weak login credentials). These attack vectors map directly to the vulnerabilities identified in Stage V. |
| **VII. Risk analysis and impact** | **1. Prepared statements / parameterized queries** — to prevent SQL injection attacks. **2. Multi-factor authentication (MFA)** — to protect against weak credentials and account takeover. **3. Encryption (AES and RSA via PKI)** — to protect sensitive data at rest and in transit. **4. Password policies and salting/hashing (SHA-256)** — to strengthen credential security and protect stored passwords. |

---

## The Seven Stages of PASTA Explained

```
Stage I   — Define Business & Security Objectives
            Understand WHY the app was built
            and what it must protect

Stage II  — Define the Technical Scope
            Identify the technologies used
            and their security implications

Stage III — Decompose the Application
            Map how data flows through the system
            (data flow diagram)

Stage IV  — Threat Analysis
            Identify threats using attacker mindset
            Internal and external threats

Stage V   — Vulnerability Analysis
            Identify weaknesses that threats
            could exploit

Stage VI  — Attack Modeling
            Build attack tree mapping how
            threats exploit vulnerabilities

Stage VII — Risk Analysis & Impact
            Recommend security controls to
            reduce the identified risks
```

---

## Technology Evaluation (Stage II Detail)

| Technology | Purpose | Security Priority |
|-----------|---------|-------------------|
| **SQL** | Stores sneaker listings, seller info, transaction data | HIGHEST — target for SQL injection, holds all sensitive data |
| **API** | Third-party integrations for added functionality | HIGH — external attack surface, potential entry point |
| **PKI (AES + RSA)** | Encrypts sensitive data and exchanges keys | HIGH — protects data confidentiality |
| **SHA-256** | Hashes passwords and credit card numbers | MEDIUM — protects stored credentials |

```
Why SQL was prioritized first:

→ It stores ALL the sensitive data
  (users, sellers, payments, transactions)

→ SQL injection is one of the most common
  and damaging attack types (OWASP Top 10)

→ A single SQL vulnerability could expose
  the entire database at once

→ The data flow diagram shows the database
  is central to the core product search process

→ Compromising SQL = compromising everything
```

---

## Data Flow Diagram Analysis (Stage III)

```
PRODUCT SEARCH PROCESS:

    User
     │
     │  "Searching for sneakers for sale"
     ▼
Product Search Process
     │
     │  SQL Query
     ▼
  Database
     │
     │  "Listings of current inventory"
     ▼
    User

SECURITY CONCERNS in this flow:
→ User input must be validated
  (prevent SQL injection)
→ Database queries must use
  prepared statements
→ Data in transit must be encrypted
→ Only authorized data should be returned
```

---

## Attack Tree Analysis (Stage VI)

```
                    USER DATA
                   (attacker goal)
                   /            \
                  /              \
         SQL Injection      Session Hijacking
              │                   │
              │                   │
      Lack of prepared      Weak login
        statements          credentials
      (vulnerability)      (vulnerability)

ATTACK PATH 1 — SQL Injection:
→ Attacker exploits lack of prepared statements
→ Injects malicious SQL code
→ Gains access to user data ⚠️

ATTACK PATH 2 — Session Hijacking:
→ Attacker exploits weak login credentials
→ Takes over a legitimate user session
→ Gains access to user data ⚠️

Both paths lead to the same goal:
COMPROMISING USER DATA
```

---

## Security Controls Recommended (Stage VII Detail)

### Control 1 — Prepared Statements / Parameterized Queries
```
ADDRESSES: SQL injection vulnerability

HOW IT WORKS:
→ Separates SQL code from user input
→ User input treated as data, never as code
→ Malicious SQL cannot be executed
→ Directly closes the SQL injection attack path
```

### Control 2 — Multi-Factor Authentication (MFA)
```
ADDRESSES: Weak login credentials / session hijacking

HOW IT WORKS:
→ Requires second verification factor
→ Stolen password alone is not enough
→ Prevents account takeover
→ Closes the session hijacking attack path
```

### Control 3 — Encryption (AES + RSA via PKI)
```
ADDRESSES: Data exposure and interception

HOW IT WORKS:
→ AES encrypts sensitive data at rest
→ RSA secures key exchange
→ Data useless to attacker even if stolen
→ Protects confidentiality throughout
```

### Control 4 — Strong Password Policies + Hashing (SHA-256)
```
ADDRESSES: Weak credentials and credential theft

HOW IT WORKS:
→ Enforce strong password requirements
→ Hash passwords with SHA-256 + salting
→ Stored passwords cannot be reversed
→ Strengthens the authentication system
```

---

## Controls Mapped to Vulnerabilities

| Vulnerability | Attack Enabled | Security Control |
|--------------|----------------|------------------|
| Lack of prepared statements | SQL injection | Prepared statements / parameterized queries |
| Weak login credentials | Session hijacking | MFA + strong password policies |
| Unencrypted data | Data interception | AES + RSA encryption (PKI) |
| Exposed stored passwords | Credential theft | SHA-256 hashing with salting |

---

## Summary

Using the PASTA framework, I conducted a comprehensive seven-stage threat model of the sneaker company's new mobile marketplace app before launch. Beginning with the business objectives  secure payment processing, data privacy, and safe account management I evaluated the app's technologies and prioritized SQL as the highest security concern because it stores all sensitive user and transaction data and is a prime target for SQL injection. Through threat analysis and the attack tree, I identified that user data could be compromised through two primary attack paths: SQL injection enabled by a lack of prepared statements, and session hijacking enabled by weak login credentials. To mitigate these risks, I recommended four security controls: prepared statements to prevent SQL injection, multi-factor authentication to prevent account takeover, AES and RSA encryption to protect data confidentiality, and strong password policies with SHA-256 hashing to secure credentials. This threat model demonstrates how proactive security analysis identifies and addresses vulnerabilities before an application is exposed to real-world attackers.

---

## Skills Demonstrated

```
✅ PASTA threat modeling framework (all 7 stages)
✅ Secure software development analysis
✅ Business and security objective alignment
✅ Technical architecture security evaluation
✅ Data flow diagram analysis
✅ Attacker mindset threat identification
✅ Vulnerability analysis (SQL injection, weak credentials)
✅ Attack tree interpretation
✅ Security control recommendation and mapping
✅ Encryption, MFA, and secure coding knowledge
✅ Professional threat model documentation
```

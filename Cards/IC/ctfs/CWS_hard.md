# Hard CTF – Full Server Compromise

During an investigation, you are given these facts:

- The web server was vulnerable to **SQL injection**
- The attacker dumped the **users table**
- One of the **passwords** cracked was reused for **SSH**
- **SSH access** was gained as a normal **user**
- **Privilege escalation** was later detected via a **kernel exploit**

---

## Question

Which **attack chain** best describes what happened?

---

## Flags (Choose One)

- **A)** XSS -> CSRF -> RCE -> Root
- **B)** SQLi -> Credential Reuse -> SSH -> Privilege Escalation
- **C)** Phishing -> Malware -> Lateral Movement
- **D)** Brute Force -> DDoS -> Defacement

---

Correct Flag: **B**

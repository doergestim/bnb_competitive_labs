# Medium CTF – Post-Exploitation

An attacker exploited a vulnerable web application and obtained a shell as the user `www-data`.

After some enumeration, they discover:

- `sudo` is installed
- A script called `backup.sh` can be run with sudo without a password
- The script calls `/bin/tar` without using a full path

---

## Question

What is the most likely next step for the **attacker**?

---

## Flags (Choose One)

- **A)** Dump the web database
- **B)** Perform a PATH hijacking attack
- **C)** Run a DDoS attack
- **D)** Deface the website

---

Correct Flag: **B**

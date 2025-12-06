![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

<img width="100" height="138" alt="BNB_CARDS_v3_02" src="https://github.com/user-attachments/assets/9675acf6-1e8d-480e-a8be-e90ea0c5fddf" />

# Compromised Web Server

A web server is considered **compromised** when an attacker manages to gain access to it without permission. This usually happens because of a vulnerability in the website, the server software, or its configuration.

Since web servers are exposed to the internet, they are one of the most common entry points used in real attacks.

---

## How Servers Usually Get Compromised

Most attacks follow a simple path:

- The attacker scans the internet for vulnerable sites
- Finds a weakness (often something outdated or misconfigured)
- Exploits it to get access
- Installs a backdoor or web shell
- Uses the server to go deeper into the network

The most common causes are:

- Old or unpatched software
- SQL injection and other input validation issues
- Unsafe file uploads
- Weak or reused passwords
- Open admin panels and bad configurations

---

## What Attackers Do After Gaining Access

Once inside, an attacker may:

- Steal databases and sensitive files
- Modify or deface the website
- Use the server to attack other systems
- Create persistence to keep access long-term
- Pivot into the internal network

At this point, the web server becomes a stepping stone for larger attacks.

---

## How Compromises Are Discovered

Compromised servers are often found through:

- Web and system log review
- SIEM alerts
- Firewall traffic analysis
- Endpoint protection warnings

In many real cases, the compromise is only discovered after damage has already been done.

---

## CTF Challenges

You will solve four challenges related to compromised web servers:

- [Easy 1 – Simple Web Exploit](ctfs/CWS_easy-1.md)
- [Easy 2 – Basic Log Investigation](ctfs/CWS_easy-2.md)
- [Medium – Post-Exploitation](ctfs/CWS_medium.md)
- [Hard – Full Server Takeover](ctfs/CWS_hard.md)

---

## Labs

Hands-on practice with the tools:

- [Burp Suite Lab](labs/burp-suite.md)
- [Caido Lab](labs/caido.md)
- [sqlmap Lab](labs/sqlmap.md)
- [Nuclei Lab](labs/nuclei.md)

---

Compromised web servers are rarely isolated incidents. They are often the first step in much larger intrusions. Learning how they are attacked and how to detect them is a core skill in defensive security.



***                                                                 
<b><i>Continuing the course? </br>[Next Card](/Cards/IC/Unauthorized_Cloud_Access.md)</i></b>

<b><i>Want to go back? </br>[Previous Card](/Cards/IC/Phishing.md)</i></b>

<b><i>Looking for a different Card? </br>[Card Directory](/card_navigation.md)</i></b>

***Finished with the Labs?***

Please be sure to destroy the lab environment!

[Click here for instructions on how to destroy the Lab Environment](/labdestruction.md)

---

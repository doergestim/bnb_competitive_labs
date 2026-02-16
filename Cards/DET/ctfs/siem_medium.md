# Medium CTF – Multi-Source Investigation

Your SIEM ingests logs from endpoint, DNS, and authentication sources.

You observe the following timeline:

```
11:03:10  AUTH   user: jsmith  login success
11:04:02  DNS    query: update-check.secure-apps.net
11:04:03  ENDPOINT  powershell.exe spawned by winword.exe
11:04:05  DNS    query: data-sync.secure-apps.net
```

The user reports only opening an email attachment.

---

## Question

What is the most likely scenario?

---

## Flags (Choose One)

- **A)** Normal software update
- **B)** Phishing attachment leading to malicious script execution
- **C)** User manually ran a maintenance script
- **D)** SIEM false positive from DNS logs

---

Correct Flag: **B**

---

# Finished?

[Next Question](siem_hard.md)

[Back to Card's Main Page](/Cards/IC/SIEM_Log_Analysis.md)

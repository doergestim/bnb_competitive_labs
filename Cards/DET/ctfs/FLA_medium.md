![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Medium CTF – Lateral Movement Detection

During an investigation, you review firewall logs from an internal segment.

You notice this sequence:

```
2024-06-10 13:20:11 ALLOW SRC=10.0.1.15 DST=10.0.2.21 DPT=445
2024-06-10 13:20:19 ALLOW SRC=10.0.1.15 DST=10.0.2.34 DPT=445
2024-06-10 13:20:26 ALLOW SRC=10.0.1.15 DST=10.0.2.56 DPT=445
2024-06-10 13:20:30 ALLOW SRC=10.0.1.15 DST=10.0.2.77 DPT=445
```

The source host normally only communicates with a file server.

---

## Question

What should you suspect first?

---

## Flags (Choose One)

- **A)** Routine file sharing activity  
- **B)** A vulnerability scan from IT  
- **C)** Lateral movement attempting SMB access  
- **D)** Time synchronization traffic

---

Correct Flag: **C**

---

# Finished?

[Next Question](FLA_hard.md)

[Back to Card's Main Page](/Cards/DET/Firewall_Log_Analysis.md)

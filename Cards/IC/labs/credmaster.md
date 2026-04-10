![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# CredMaster

# Ubuntu VM

---

## What is CredMaster?

**CredMaster** is a password spraying framework used by penetration testers and red teams. Unlike brute-force attacks that try every password against one account, **password spraying** tries one or a few common passwords against many accounts - slowly - to avoid account lockouts.

CredMaster is built around **plugins** that target different authentication services (Office 365, Okta, AWS, VPNs, etc.) and can optionally distribute requests through AWS Lambda to rotate IP addresses. In this lab we skip the cloud rotation and spray against a local server we control.

---

### Spin up the Mock-up server meant for the Password Spray

Open up a terminal

```bash
cd ~/BnB/credMaster
```

```bash
python3 mock_auth_server.py
```

<img width="705" height="116" alt="2026-04-09_12-40" src="https://github.com/user-attachments/assets/afa7dfc5-4373-4d75-9027-6c1c7700250f" />

---

## Open the Defender View

Before spraying, open a **third terminal** to watch the server logs live - this is the blue team / analyst perspective:

```bash
tail -f /tmp/auth_server.log
```

Leave this running. Every login attempt will appear here in real time as CredMaster works through the lists.

---

## Run the Password Spray

You now have three terminals running:
- **Terminal 1** - the mock auth server (target)
- **Terminal 2** - the live log watcher (defender)
- **Terminal 3** - ready for CredMaster (attacker)


Before running the spray we need to create an **AWS Account**, go here and make an account: https://aws.amazon.com/

<img width="785" height="135" alt="2026-04-09_13-24" src="https://github.com/user-attachments/assets/edb3cf0a-f47a-41d1-b92c-b1303fac1051" />

Afert you have made an account, go here: https://us-east-1.console.aws.amazon.com/iam/home?region=eu-north-1#/users and create a new user

<img width="581" height="240" alt="2026-04-09_14-13" src="https://github.com/user-attachments/assets/98b39c9e-d0f3-4d59-a557-2e0f0015eb10" />


<img width="742" height="340" alt="2026-04-09_14-14" src="https://github.com/user-attachments/assets/16c022dc-ea0d-4d09-a68c-895847e037ee" />

<img width="1438" height="688" alt="2026-04-09_14-15" src="https://github.com/user-attachments/assets/ae4f4352-0fc6-4676-8444-d1c749c5c49b" />

After creating the user with the settings from above, you should see this:

<img width="705" height="283" alt="2026-04-09_14-17" src="https://github.com/user-attachments/assets/4ec235cb-b54b-4b6f-8681-7f89cf3193c7" />

Click on the user and then click `Create access key`

<img width="1615" height="815" alt="2026-04-09_14-19" src="https://github.com/user-attachments/assets/159e0f4b-7187-4f76-ba92-8152b6287d75" />

Select `Command Line Interface (CLI)` and click though the confirmation

<img width="1213" height="778" alt="2026-04-09_14-20" src="https://github.com/user-attachments/assets/3873d292-319d-4854-953b-dad487f0477f" />

Now this is REALLY IMPORTANT, save the **Access Key** and the **Secret access key**

<img width="1155" height="526" alt="2026-04-09_14-21" src="https://github.com/user-attachments/assets/80458a46-8a6b-446b-af9c-83564e7bc372" />



In Terminal 3, run the spray with your AWS Keys:

```bash
cd ~/BnB/credMaster/CredMaster
```

```bash
source venv/bin/activate
```

```bash
python3 credmaster.py \
  --plugin httppost \
  -u /usr/share/seclists/Usernames/Names/names.txt \
  -p /usr/share/seclists/Passwords/Common-Credentials/top-passwords-shortlist.txt \
  --access_key YOUR_AWS_KEY \
  --secret_access_key YOUR_AWS_SECRET \
  --pluginargs url http://127.0.0.1:5000/auth/login content-type json \
  --delay 1 \
  --jitter 500
```

**What these flags do:**

| Flag | Meaning |
|------|---------|
| `--plugin localtest` | Use our custom plugin |
| `-u` | SecLists username file |
| `-p` | SecLists password file |
| `--nofire` | Skip AWS Lambda, run locally |
| `--delay 1` | Wait 1 second between attempts |
| `--jitter 500` | Add up to 500ms of random extra delay |

> The `--delay` and `--jitter` flags are critical in real engagements - they mimic human timing to stay under lockout thresholds. They also make the output easier to follow here.

Watch **both perspectives** as it runs:

**Attacker terminal** - CredMaster will find multiple hits since 15 accounts all share the same weak password:
```
[+] VALID CREDENTIALS: aaron:Password1
[+] VALID CREDENTIALS: admin:Password1
[+] VALID CREDENTIALS: alex:Password1
[+] VALID CREDENTIALS: alice:Password1
...
```

**Defender terminal** - the log shows every attempt in real time:
```
2024-10-15 14:23:01 - [FAILED]  LOGIN ATTEMPT | IP: 127.0.0.1 | User: root | Pass: 123456
2024-10-15 14:23:02 - [FAILED]  LOGIN ATTEMPT | IP: 127.0.0.1 | User: aaron | Pass: 123456
2024-10-15 14:23:03 - [FAILED]  LOGIN ATTEMPT | IP: 127.0.0.1 | User: adam | Pass: 123456
...
2024-10-15 14:25:12 - [SUCCESS] LOGIN ATTEMPT | IP: 127.0.0.1 | User: aaron | Pass: Password1
2024-10-15 14:25:44 - [SUCCESS] LOGIN ATTEMPT | IP: 127.0.0.1 | User: admin | Pass: Password1
2024-10-15 14:26:01 - [SUCCESS] LOGIN ATTEMPT | IP: 127.0.0.1 | User: alice | Pass: Password1
```

The pattern is unmistakable from the defender's view: the **same IP cycling through hundreds of usernames, one password at a time, and hitting multiple accounts**. That is the signature of a spray - and exactly why a single weak password policy can compromise an entire organisation at once.

---

## View the Results

After the run completes, check what CredMaster saved:

```bash
cat ~/CredMaster/found_credentials.txt
```

You should see all 15 compromised accounts listed - every user who had `Password1` as their password.

View the complete log:

```bash
cat /tmp/auth_server.log
```

Count total attempts made:

```bash
wc -l /tmp/auth_server.log
```

---

## Detection Exercises

Now think like a defender. Run these against the log:

**How many unique usernames were targeted?**
```bash
grep "LOGIN ATTEMPT" /tmp/auth_server.log | grep -oP 'User: \K[^\s|]+' | sort -u | wc -l
```

**Which passwords were tried most often?**
```bash
grep "LOGIN ATTEMPT" /tmp/auth_server.log | grep -oP 'Pass: \K.*' | sort | uniq -c | sort -rn | head
```

**How many failed attempts happened before the successful one?**
```bash
grep -c "FAILED" /tmp/auth_server.log
```

**What was the time window of the entire attack?**
```bash
head -1 /tmp/auth_server.log && tail -1 /tmp/auth_server.log
```

> **Defender insight:** A SIEM rule that fires on "more than 10 failed logins from the same IP across more than 5 different accounts within 10 minutes" would catch this spray immediately. The delay and jitter slow it down but do not change the underlying pattern.










---

# Finished?

[Back to Card's Main Page](/Cards/IC/External_Password_Spray.md)

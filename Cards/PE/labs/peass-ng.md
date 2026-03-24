![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# PEASS-ng

# Ubuntu VM

**Goal:** Learn what **PEASS-ng / LinPEAS** can discover on a Linux host by **creating a few safe “bad configs” in a lab VM**, running LinPEAS, and reviewing the findings.  

https://github.com/peass-ng/PEASS-ng

---

## What you’ll do in this lab

- Generate a report and filter for “interesting” lines
- Create 3 simple lab misconfigurations and see LinPEAS detect them:
  1. **World-readable “app config” with credentials**
  2. **A root cron job that runs a script**
  3. **A cron script that is writable by normal users** (a classic “this is bad” finding)
- Fix the misconfigurations and re-run LinPEAS to confirm improvement



---

## Create 3 simple lab misconfigurations

### Create a world-readable “app config” with fake credentials

```bash
sudo mkdir -p /opt/demoapp
```

```bash
sudo bash -c 'cat > /opt/demoapp/config.ini << "EOF"
[database]
host=127.0.0.1
user=demoapp
password=SuperFakePassword123!
EOF'
```

Make it readable by everyone (bad):

```bash
sudo chmod 644 /opt/demoapp/config.ini
```

```bash
ls -l /opt/demoapp/config.ini
```

Quick check as `ubuntu`:

```bash
cat /opt/demoapp/config.ini
```

<img width="333" height="82" alt="image" src="https://github.com/user-attachments/assets/54a49b96-5051-40b6-90d5-03e2766a6b4f" />


---

### Create a cron job that runs as root every minute

Create a script that the cron job will run:

```bash
sudo bash -c 'cat > /usr/local/bin/demo-backup.sh << "EOF"
#!/bin/bash
# Demo script: writes a timestamp to a log file
date >> /var/log/demo-backup.log
EOF'
```

```bash
sudo chmod +x /usr/local/bin/demo-backup.sh
```

Create a **root cron entry**:

```bash
sudo bash -c 'cat > /etc/cron.d/demo-backup << "EOF"
* * * * * root /usr/local/bin/demo-backup.sh
EOF'
```

Wait a minute, then confirm it’s executing:

```bash
sudo tail -n 5 /var/log/demo-backup.log
```

<img width="272" height="22" alt="image" src="https://github.com/user-attachments/assets/a14f44e9-4c77-42ae-85a1-cdba03194e0d" />


---

### Make the root cron script writable by normal users (intentionally bad)

This is the “obvious problem” we want LinPEAS to scream about:

```bash
sudo chmod 777 /usr/local/bin/demo-backup.sh
```

```bash
ls -l /usr/local/bin/demo-backup.sh
```

<img width="644" height="21" alt="image" src="https://github.com/user-attachments/assets/19f3d808-8261-480d-8e62-dc1517a64616" />


> A root cron job + a writable script is a serious misconfiguration.
---

## Run LinPEAS

```bash
cd ~/BnB/peass-ng
```

```bash
./linpeas.sh | tee linpeas-after.txt
```

>[!NOTES]
>Bare in mind it can take up to 10+ minutes

You’ll likely see some normal system findings (that’s expected). Important are the few **very obvious** misconfigs that we added

### Find the “demoapp credentials” finding

```bash
grep -nE "/opt/demoapp/config.ini|demoapp|SuperFakePassword" linpeas-after.txt
```

### Find cron-related findings

```bash
grep -nEi "cron|/etc/cron.d|demo-backup|demo-backup.sh|writable" linpeas-after.txt | head -n 80
```

<img width="1066" height="355" alt="image" src="https://github.com/user-attachments/assets/23dc6600-1076-41c9-a968-ccae1805835c" />


- We can see with red on a yellow backround the REALLYY **bad stuff** - which is the cron job we just made

### Find “writable root-owned files” findings

```bash
grep -nEi "writable.*root|root.*writable" linpeas-after.txt | head -n 80
```

<img width="701" height="27" alt="image" src="https://github.com/user-attachments/assets/549cd7a4-42b7-4eb8-8190-55683038dee3" />

We can see it found something, it's over at the **5543** line, let's see what it found

```bash
sed -n '5543,5600p' linpeas-after.txt
```

<img width="985" height="369" alt="image" src="https://github.com/user-attachments/assets/68681101-b838-4bcd-950f-92629788b020" />


---

## What LinPEAS is showing you (quick tour)

Open the output file in a text editor:

```bash
nano linpeas-after.txt
```

As you scroll, look for sections like:

- **System information**: kernel, distro, container/VM hints
- **Users & groups**: who exists, who is in sudo/admin groups
- **Sudo checks**: what commands a user can run with sudo, environment variables, etc.
- **Cron jobs & timers**: scheduled tasks (like our `/etc/cron.d/demo-backup`)
- **Interesting/writable files**: world-writable paths, weak permissions
- **Credentials**: files containing `password=`, tokens, SSH keys, config files
- **Network and processes**: services and ports that might matter

---

## Fix the misconfigurations (defender mode)

### Fix the world-readable config file

Restrict it to root only:

```bash
sudo chmod 600 /opt/demoapp/config.ini
```

```bash
sudo chown root:root /opt/demoapp/config.ini
```

```bash
ls -l /opt/demoapp/config.ini
```

### Fix the writable cron script

Make it only writable by root:

```bash
sudo chmod 755 /usr/local/bin/demo-backup.sh
```

```bash
sudo chown root:root /usr/local/bin/demo-backup.sh
```

```bash
ls -l /usr/local/bin/demo-backup.sh
```

### Remove the demo cron job entirely

```bash
sudo rm -f /etc/cron.d/demo-backup
```

---

## Re-run LinPEAS to verify fixes

```bash
cd ~/BnB/peass-ng
```

```bash
./linpeas.sh | tee linpeas-fixed.txt
```

Now re-run the same filters and compare:

```bash
echo "=== After (insecure) ==="
grep -nEi "demoapp|demo-backup|/etc/cron.d/demo-backup|SuperFakePassword|writable" linpeas-after.txt | head -n 80
```

<img width="1149" height="424" alt="image" src="https://github.com/user-attachments/assets/a4f87d19-3de5-4c9d-9631-e914d0ab4682" />


```bash
echo "=== Fixed ==="
grep -nEi "demoapp|demo-backup|/etc/cron.d/demo-backup|SuperFakePassword|writable" linpeas-fixed.txt | head -n 80
```

<img width="1151" height="379" alt="image" src="https://github.com/user-attachments/assets/bc78fbbc-7961-4fd2-aaa9-29eaa073df03" />


You should see those “obvious” findings disappear or become less severe.

---

## Cleanup

```bash
sudo rm -rf /opt/demoapp
sudo rm -f /usr/local/bin/demo-backup.sh
sudo rm -f /var/log/demo-backup.log
sudo rm -f /etc/cron.d/demo-backup
```

---

# Finished?

[Back to Card's Main Page](/Cards/PE/Local_Privilege_Escalation.md)

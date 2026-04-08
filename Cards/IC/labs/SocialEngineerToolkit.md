![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Social Engineering Toolkit (SET)

# Ubuntu VM

---

## In this lab we will

- Clone a real website to create a credential harvester
- Send a simulated phishing payload using SET's built-in tools
- Understand how attackers think - so defenders can stop them

---

## What is SET?

The **Social Engineering Toolkit (SET)** is an open-source penetration testing framework designed specifically for **social engineering attacks**. It was created by TrustedSec and comes pre-installed on Kali Linux. It automates attacks like:

- **Credential Harvesting** - cloning websites to steal logins
- **Phishing Emails** - crafting convincing fake emails
- **Payload Generation** - creating malicious files
- **QR Code Attacks** - embedding malicious links in QR codes

---

## Launching SET

- Launch the toolkit:

```bash
sudo setoolkit
```

<img width="687" height="682" alt="2026-04-08_12-06" src="https://github.com/user-attachments/assets/8c80bf20-c5be-4db9-8c55-773041576a27" />


This is the SET main menu, everything is **number-driven** - you just type the number and press Enter

---

## Credential Harvester Attack (Website Cloning)

This is one of SET's most well-known features. It clones a real website and captures any credentials typed into it - all running **on your own machine**.

### Step 1 - Enter Social Engineering Attacks

```
set> 1
```

You will see:

<img width="484" height="373" alt="2026-04-08_12-08" src="https://github.com/user-attachments/assets/d7640b4f-8401-4c0d-b159-337a2035e510" />


### Select Website Attack Vectors

```
set> 2
```

You will see:

<img width="998" height="650" alt="2026-04-08_12-10" src="https://github.com/user-attachments/assets/90efe03d-a604-4051-917a-2944f88fb155" />


### Select the Credential Harvester Attack Method

```
set:webattack> 3
```

You will see:

<img width="711" height="417" alt="2026-04-08_12-11" src="https://github.com/user-attachments/assets/2ccba603-79ff-4f8f-bb5b-771259e44fdd" />


### Use the Site Cloner

```
set:webattack> 2
```

SET will ask for your IP address (where the fake site will be hosted):

```
set:webattack> IP address for the POST back in Harvester/Tabnabbing:
```

Type your local machine IP. To find it, open **another terminal** and run:

```bash
ip a | grep "inet " | grep -v 127
```

Use the IP shown (usually something like `192.168.x.x`). Type it and press Enter

### Enter the site to clone

SET will ask which URL to clone:

```
set:webattack> Enter the url to clone: http://testphp.vulnweb.com/login.php
```

> We are cloning **testphp.vulnweb.com** - this is a **deliberately vulnerable test site** created by Acunetix specifically for security testing. It is safe and legal to use

SET will clone the site and start a web server on port **80**.

You will see:

```
[*] Cloning the website: http://testphp.vulnweb.com/login.php
[*] This could take a little bit...
[*] Harvester is ready, have victim go to your site.
[*] Saving harvested credentials to /var/www/...
```

### Open the fake site in your browser

Open your browser and go to:

```
http://127.0.0.1
```

You will see a **pixel-perfect clone** of the login page. Go ahead and type in some fake credentials:

- **Username:** `testuser`
- **Password:** `Password123`

Click the login button.

### Watch SET capture the credentials

Go back to your terminal where SET is running. You will see the captured data printed in **real time**:

```
[*] WE GOT A HIT! Printing the output:
POSSIBLE USERNAME FIELD FOUND: tfUName=testuser
POSSIBLE PASSWORD FIELD FOUND: tfUPass=Password123
[*] WHEN YOUR FINISHED, HIT CONTROL-C TO GENERATE A FULL REPORT.
```

SET captured everything typed into the fake form - **username, password, and more** - without the "victim" knowing anything happened

### Stop the harvester

Press `CTRL+C` to stop the server. SET will automatically generate a report saved to:

```bash
/root/.set/reports/
```

View it:

```bash
ls /root/.set/reports/
cat /root/.set/reports/*.xml
```

---

## QR Code Attack Generator

SET can generate a malicious QR code that redirects a victim to a URL of your choice (like a phishing page or payload)

### Step 1 - Go back to the main menu

```
set> 1
```

```
set:attackvectors> 9
```

> If option 9 is not visible, scroll down - menu options vary slightly by SET version

### Select QR Code Attack

```
set:qrcode> 1
```

SET will ask for the URL to embed:

```
set:qrcode> Enter the URL you want the QR code to go to: http://127.0.0.1
```

SET will generate a QR code image saved to your machine:

```bash
ls /root/.set/qrcode*
```

Open it to see the generated QR code:

```bash
eog /root/.set/qrcode.png
```

> **What this demonstrates:** Attackers print or share these QR codes in public (flyers, emails, posters) to redirect victims to phishing pages - bypassing URL suspicion entirely since people do not "read" a QR code

---

## Exploring the Spear Phishing Menu

Go back to the main menu:

```
set> 1
set:attackvectors> 1
```

You will see options like:

```
   1) Perform a Mass Email Attack
   2) Create a FileFormat Payload
   3) Create a Social-Engineering Template
```

- Option **1** lets you send a phishing email with an attached payload to a target
- Option **2** lets you embed payloads in `.pdf`, `.docx`, `.xlsx` files
- Option **3** lets you craft a custom email template

**This is how real attackers operate.** A crafted email + a weaponized attachment = a convincing spear phishing campaign that bypasses most user awareness

Type `99` or press `CTRL+C` to exit without sending anything

---

## Checking SET Logs

SET logs everything it does. Let's look at what was recorded during our session:

```bash
ls /root/.set/
```

```bash
cat /root/.set/set.log
```

You will see timestamped entries for every action taken - cloning, credential captures, payload generation, and more. This is the attacker's trail - in a real investigation, forensic analysts look for exactly these artifacts

---

## Understanding the Defender's View

Now let's think like a **defender**. What would you see on the network during a credential harvester attack?

Open a new terminal and run a packet capture **before** visiting the fake site:

```bash
sudo apt install -y tcpdump
sudo tcpdump -i lo -w /tmp/set_capture.pcap port 80
```

Now revisit `http://127.0.0.1`, fill in fake credentials, then stop the capture with `CTRL+C`

Open the capture:

```bash
sudo apt install -y tshark
tshark -r /tmp/set_capture.pcap -Y "http.request.method == POST" -T fields -e http.file_data
```

You will see the raw POST body containing the credentials - **in plain text** - exactly as a network analyst or IDS would detect it on an unencrypted connection














---

# Finished?

[Back to Card's Main Page](/Cards/IC/Phishing.md)

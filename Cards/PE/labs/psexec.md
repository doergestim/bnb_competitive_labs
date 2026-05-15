![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# PsExec

## In this lab we will

- Use a **Linux VM as the attacker**
- Use a **Windows VM as the target**
- Run **PsExec-style remote execution** with Impacket `psexec.py`
- Execute commands on the Windows target over SMB
- Understand how PsExec-style execution works internally
- Observe service creation artifacts and Windows logs
- Practice basic detection thinking

> [!IMPORTANT]
> This lab uses **Impacket PsExec**, not Microsoft Sysinternals `PsExec.exe`.
> The technique is practicaly the same: remote execution over SMB using administrative credentials and temporary service creation.

---

# Lab Topology

```text
Linux VM     →     Windows VM
Attacker           Target
```

Example:

```text
Linux attacker IP : 192.168.56.20
Windows target IP : 192.168.56.30
```

---

# Overview

PsExec-style execution works by:

1. Connecting to the Windows target over **SMB**
2. Authenticating with administrative credentials
3. Uploading a temporary service binary
4. Creating and starting a service
5. Running commands through that service
6. Returning a remote command shell

---

# Secure Setup

Before running remote execution, restrict which machine can connect to the Windows target.

# Part 1 — Limit allowed IPs on Windows Firewall

On the **Windows target**:

1. Open **Control Panel**
2. Click **System and Security**
3. Go to **Windows Defender Firewall**
4. Click **Advanced Settings**
5. Go to **Inbound Rules**
6. Find rules related to:
  - `File and Printer Sharing (SMB-In)`
6. Open the rule (with double click)
7. Go to the **Scope** tab
8. Under **Remote IP address**, select:
  - **These IP addresses**
9. Add only the **Linux attacker IP**

This limits SMB access to your attacker VM only.

<img width="2223" height="1168" alt="img1" src="https://github.com/user-attachments/assets/7eb188e9-40d7-4aba-8305-aa2be181fd09" />

---

## Enable local admin remote access

When using a local administrator account remotely, Windows may filter the admin token.  
Enable full local admin token access for this lab, run this command in powershell:

```powershell
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```

---

# Part 2 — Connectivity Tests

Open an Ubuntu terminal:

<img width="56" height="59" alt="img08" src="https://github.com/user-attachments/assets/a80b538b-bf3e-4155-b2f6-e7da26cd663f" />

```bash
cd ~/BnB/psexec-lab/
```

Replace `<TARGET_IP>` with the Windows target IP.

## Ping the target

```bash
ping -c 3 <TARGET_IP>
```

---

## Check SMB port 445

```bash
nc -vz <TARGET_IP> 445
```

You should see that port `445` is open.

<img width="516" height="223" alt="img2" src="https://github.com/user-attachments/assets/585ead06-d44f-4042-943a-5a211669be62" />

---

## List SMB shares

```bash
smbclient -L //<TARGET_IP> -U 'labadmin%Password123!'
```

If this succeeds, the Linux attacker can authenticate to the Windows target over SMB.

---

# Part 3 — Basic Impacket PsExec Usage

## Open a remote shell

In the Linux terminal, run the following commands:

```bash
source venv/bin/activate
```

```bash
psexec.py 'labadmin:Password123!@<TARGET_IP>'
```

If successful, you should receive a Windows command shell.

<img width="832" height="257" alt="img3" src="https://github.com/user-attachments/assets/391f2b93-ee33-4942-b64c-721209d3f1f2" />

---

## Run basic commands inside the remote shell

Inside the shell, run:

```cmd
whoami
hostname
ipconfig
```

You should see that commands are executing on the **Windows target**, not on Linux.

---

# Part 4 — Use a Predictable Service Name

By default, Impacket may use a temporary service name.  
For lab analysis, use a predictable service name.

Exit the current shell, then run:

```bash
psexec.py -service-name LabPSEXESVC -remote-binary-name LabPSEXESVC.exe 'labadmin:Password123!@<TARGET_IP>'
```

Inside the shell:

```cmd
whoami
hostname
```

This makes the service easier to find in logs and artifact analysis.

>[!NOTE]
>
>Don't exit the psexec session, we need it for the next part!

---

# Part 5 — Artifact Analysis on the Windows Target

Now inspect what happened on the **Windows target**.

## Check for the service

Run on Windows in CMD:

```cmd
sc query LabPSEXESVC
```

<img width="536" height="254" alt="cmd_service" src="https://github.com/user-attachments/assets/7a889445-3ac2-43cb-8f7b-845439210323" />

or in PowerShell:

```powershell
Get-Service LabPSEXESVC -ErrorAction SilentlyContinue
```

<img width="638" height="232" alt="powershell_service" src="https://github.com/user-attachments/assets/ea28aa8b-8832-498b-a4ff-391d3d0a84f6" />

---

## Check Event Viewer

Open:

```text
Event Viewer → Windows Logs → System
```

Look for:

- Service Control Manager events
- Event ID `7045`
- service creation involving `LabPSEXESVC`

You can filter the logs, using 'Filter Current Log...' option

<img width="1918" height="907" alt="graphrunner66" src="https://github.com/user-attachments/assets/e89ba267-dc3f-4800-b57e-6e0bcd8174b7" />

---

# Part 6 — Run Single Commands

You can also run a specific command instead of working interactively.

```bash
psexec.py 'labadmin:Password123!@<TARGET_IP>' whoami
```

```bash
psexec.py 'labadmin:Password123!@<TARGET_IP>' hostname
```

```bash
psexec.py 'labadmin:Password123!@<TARGET_IP>' ipconfig
```

This is useful for quick remote checks.

---

# Part 7 — Useful Impacket PsExec Options

## Use hashes instead of a password

If you have an NTLM hash, you can authenticate without the plaintext password.

To get an NTLM hash you can use one of the online password hashers, for examle: https://www.browserling.com/tools/ntlm-hash

```bash
psexec.py -hashes :<NTLM_HASH> 'labadmin@<TARGET_IP>'
```

---

## Use Kerberos authentication

In domain environments, Kerberos can be used instead of NTLM.

> [!WARNING]
> This lab is not prepared for running this command

```bash
psexec.py -k -no-pass 'DOMAIN/labadmin@<TARGET_HOSTNAME>'
```

This requires a valid Kerberos ticket.

---

# Part 8 — Detection Thinking

PsExec-style activity can be detected through:

- SMB connections to port `445`
- remote authentication as a local or domain admin
- temporary service creation
- suspicious service names
- service binaries written to `C:\Windows`
- remote command shells
- Event ID `7045`

A defender should correlate:

```text
SMB logon → Admin share access → Service creation → Process execution
```

---

# Finished?

[Back to Card's Main Page](/Cards/PE/New_Service_Creation-Modification.md)

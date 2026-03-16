![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# SharpUp

# Windows VM

## In this lab we will

- Download **SharpUp** using PowerShell
- Create **two realistic privilege‑escalation misconfigurations**
- Run **SharpUp** to audit the system
- Save the output to a file
- Locate the misconfigurations in the SharpUp results

The goal is to understand **how SharpUp detects privilege‑escalation issues**.

---

# Setup

## Create the lab directory

Open **PowerShell as Administrator** and input all of the following commands there.

```powershell
mkdir C:\SharpUpLab -ErrorAction SilentlyContinue
cd C:\SharpUpLab
```

---

# Download SharpUp

Download the lab archive directly from GitHub.

```powershell
Invoke-WebRequest `
  -Uri "https://github.com/doergestim/bnb_competitive_labs/raw/main/FilesForLabs/Sharp.zip" `
  -OutFile Sharp.zip
```

Extract the archive:

```powershell
Expand-Archive Sharp.zip -DestinationPath .
```

Verify:

```powershell
dir
```

You should see .zip and .exe files in the directory:

![image](/FilesForLabs/images/sharpUp1.png)

---

# Create Misconfiguration 1 — Autorun entry pointing to a writable path

This misconfiguration creates a **startup program entry** in the Windows registry that points to an executable located in a directory writable by normal users.  
If the executable can be modified by non‑administrative users, it may allow **privilege escalation or persistence**.

Create a directory that normal users will be able to modify.

```powershell
mkdir C:\SharpUpLab\AutorunApp -ErrorAction SilentlyContinue
```

Grant **Users** modify permissions.

```powershell
icacls "C:\SharpUpLab\AutorunApp" /grant "Users:(OI)(CI)M"
```

Create a simple dummy executable file.

```powershell
Set-Content C:\SharpUpLab\AutorunApp\autorunapp.exe "test"
```

Create the Autorun registry entry

Add the application to the Windows **Run** key so it executes at login.

```powershell
reg add "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" `
 /v VulnAutorunApp `
 /t REG_SZ `
 /d "C:\SharpUpLab\AutorunApp\autorunapp.exe" `
 /f
```

Verify the entry:

```powershell
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"
```

You should see:

![image](/FilesForLabs/images/sharpUp2.png)

---

# Create Misconfiguration 2 — Writable service binary

Create a test service directory.

```powershell
mkdir C:\SharpUpLab\Service
```

Create a dummy service executable file.

```powershell
echo test > C:\SharpUpLab\Service\labservice.exe
```

Grant normal users modify permissions:

```powershell
icacls "C:\SharpUpLab\Service\labservice.exe" /grant "Users:(M)"
```

Create a Windows service pointing to this binary.

```powershell
sc.exe create SharpUpLabService binPath= "C:\SharpUpLab\Service\labservice.exe" start= auto
```

Verify the service:

```powershell
sc.exe qc SharpUpLabService
```

Expected output:

![image](/FilesForLabs/images/sharpUp3.png)

---

# Run SharpUp

Generate the audit and save it to a file.

```powershell
.\SharpUp.exe audit | Tee-Object -FilePath sharpup-output.txt
```

> [!NOTE]
> Wait until the end of the audit.

---

# Locate the misconfigurations

Search the output for the issues you created.

```powershell
Select-String -Path sharpup-output.txt -Pattern "VulnAutorunApp|autorunapp.exe|SharpUpLabService|labservice.exe"
```

You should find:

![image](/FilesForLabs/images/sharpUp4.png)

---

# Open audit file

Now, after finishing these tasks, you can take a look at the rest of the **sharpup-output.txt** file. The audit mode was run in high integrity, so many of the findings are false-positives.

```powershell
notepad sharpup-output.txt
```

![image](/FilesForLabs/images/sharpUp5.png)

Anayze the list of different vulnerabilities found by **SharpUp**.
Scroll through the output and review the findings. SharpUp reports potential privilege escalation issues in several categories, including:

* Modifiable Service Binaries

* Services with Modifiable Registry Keys

* Unquoted Service Paths

* Modifiable Scheduled Tasks

* Registry Autorun Entries

* Modifiable Folders in PATH

---

# Finished?

[Back to Card's Main Page](/Cards/PE/Local_Privilege_Escalation.md)

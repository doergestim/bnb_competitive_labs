![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Seatbelt

# Windows VM

## In this lab we will

- Download **Seatbelt**
- Create interesting system artifact
- Check logon events in the past 30 days
- Run specific Seatbelt enumeration groups
- Analyze the results

## Seatbelt

Seatbelt is a command-line tool from the GhostPack project used for **system enumeration and situational awareness on Windows systems**. It gathers a wide range of information about the operating system, users, services, registry, environment variables, and other system components.

Unlike exploitation tools, Seatbelt does not attempt to gain privileges. Instead, it helps identify **potential security weaknesses, misconfigurations, and sensitive data exposures** that could be used during a security assessment.

Seatbelt organizes its findings into structured sections and supports running either **targeted checks or grouped modules**, making it flexible for both quick reconnaissance and deeper analysis.

It is commonly used by penetration testers and red teamers to quickly understand a system and identify areas that may require further investigation.

# Setup

## Go to the Lab Directory

Open **PowerShell as Administrator**.

```powershell
mkdir C:\SeatbeltLab -ErrorAction SilentlyContinue
cd C:\SeatbeltLab
```

## Download Seatbelt

```powershell
Invoke-WebRequest `
  -Uri "https://github.com/doergestim/bnb_competitive_labs/raw/main/FilesForLabs/Seatbelt.zip" `
  -OutFile Seatbelt.zip
```

Extract:

```powershell
Expand-Archive Seatbelt.zip -DestinationPath .
```

Verify:

```powershell
dir
```

You should see .zip and .exe files in the directory:

<img width="670" height="257" alt="seatbelt1" src="https://github.com/user-attachments/assets/5f6ae62b-ddbe-48fb-8c41-00a1a30a1c0a" />

# Create Lab Artifact

## Artifact — Suspicious environment variable

Create a custom environment variable containing sensitive data.

```powershell
setx API_KEY "SECRET-KEY-12345"
```

# Run Seatbelt

## 1. Logon events (recent activity)

```powershell
.\Seatbelt.exe "LogonEvents 30"
```

This command shows recent logon events from the last 30 days, which can reveal:

- User activity
- Login patterns
- Potential lateral movement

<img width="1201" height="593" alt="seatbelt2" src="https://github.com/user-attachments/assets/915ee1dc-8911-4eeb-a446-9056d544087e" />

> [!WARNING]
> Your output might be different depending on your activity on the vm.

## 2. User-focused checks

```powershell
.\Seatbelt.exe -group=user -outputfile="user-output.txt"
```

## 3. System-focused checks

```powershell
.\Seatbelt.exe -group=system -outputfile="system-output.txt"
```

## 4. File / interesting data checks

```powershell
.\Seatbelt.exe -group=misc -outputfile="misc-output.txt"
```

# Analyze the Output

Open the files and familiarize yourself with the content of Seatbelt output files:

```powershell
notepad user-output.txt
notepad system-output.txt
notepad misc-output.txt
```

Example part of the `misc-output.txt` file:

<img width="1896" height="812" alt="seatbelt3" src="https://github.com/user-attachments/assets/b5363de5-dc90-45ff-b6b3-15c9e47a8645" />

## What to look for

Seatbelt output is divided into sections. Important ones include:

- **Environment Variables**  
  Displays system and user environment variables. These can contain sensitive information such as API keys, tokens, file paths, or configuration data that may be useful during a security assessment.

- **Interesting Files**  
  Lists files in commonly sensitive locations (e.g., user directories, temp folders). This can reveal scripts, configuration files, or documents that may contain credentials or other valuable data.

- **Installed Applications / Program Files**  
  Shows installed software and directories under locations like `Program Files`. This helps identify outdated or vulnerable applications, as well as potential locations of sensitive configuration files.

- **User Data Locations**  
  Highlights directories where user-specific data is stored (e.g., Desktop, Documents, AppData). These locations often contain files that may expose credentials, tokens, or personal information.

# Identify the Artifact

You created an artifact, locate it in the Seatbelt output.

Search for keyword:

```powershell
Select-String *.txt -Pattern "SECRET-KEY-12345"
```

<img width="1200" height="117" alt="seatbelt4" src="https://github.com/user-attachments/assets/1c7ebf42-7297-49a7-8b2d-ca85bd0e9baa" />

The key is present in the system-output file.

---

# Finished?

[Back to Card's Main Page](/Cards/PE/Local_Privilege_Escalation.md)

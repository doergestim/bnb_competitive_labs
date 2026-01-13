![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Nuclei



## Lab Goal

This lab introduces **Nuclei**, a fast vulnerability scanner based on templates.
You will:
- Deploy OWASP Juice Shop (intentionally vulnerable app)
- Run basic Nuclei scans
- Understand results from both attacker and defender perspectives

This lab is beginner-friendly and fully hands-on.

---

## Install Docker

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

Verify Docker:
```bash
docker --version
```

---

## Run OWASP Juice Shop

Pull and run Juice Shop:
```bash
sudo docker run -d -p 3000:3000 bkimminich/juice-shop:v10.3.0
```

Verify it is running:
```bash
sudo docker ps
```

<img width="1550" height="70" alt="image" src="https://github.com/user-attachments/assets/378b9008-4c54-4860-a843-5eef325a2cb3" />


Open your browser and go to:
```
http://localhost:3000
```

<img width="1915" height="1050" alt="image" src="https://github.com/user-attachments/assets/43bcd849-e417-46c8-a831-c661f47fe6e0" />


You should see the OWASP Juice Shop homepage

---

## Install Nuclei

Install Go (required):
```bash
sudo apt install -y golang
```

Install Nuclei:
```bash
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```

Add Go binaries to PATH:
```bash
echo 'export PATH=$PATH:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
```

Verify installation:
```bash
nuclei -version
```

---

## Download Nuclei Templates

Nuclei uses templates to detect vulnerabilities.

```bash
nuclei -update-templates
```

Templates are stored in:
```bash
~/.local/nuclei-templates
```

---

## First Scan (Basic Detection)

Run a simple scan against Juice Shop:
```bash
nuclei -u http://localhost:3000
```

Observe:
- Detected technologies
- Exposed endpoints
- Informational findings

<img width="1920" height="842" alt="image" src="https://github.com/user-attachments/assets/49a53167-c44c-4862-82dc-dcb2ebd3ad9c" />

This is passive recon-style scanning

---

## Run Vulnerability Templates

Now run only vulnerability-related templates:
```bash
nuclei -u http://localhost:3000 -tags vuln
```

You may see:
- Missing security headers
- Misconfigurations
- Known vulnerable patterns

---

## Filter a scan to only show specific severities

```bash
nuclei -u http://localhost:3000 -severity info
```

---

## Save Scan Results

Run scan and save output:
```bash
nuclei -u http://localhost:3000 severity info -o results.txt
```

View results:
```bash
cat results.txt
```

This is how defenders:
- Review **findings**
- Prioritize **risks**
- Feed **SIEM** / **reports**



---

# Finished?

[Back to Card's Main Page](/Cards/IC/External_Service_Exploitation.md)

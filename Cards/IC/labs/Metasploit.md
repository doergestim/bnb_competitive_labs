![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Metasploit


## In this lab we will
- Install Metasploit Framework (simple, standard install)
- Learn the Metasploit console basics (`search`, `use`, `info`, options)
- Create a **safe local target** service (an FTP server on localhost)
- Use Metasploit **scanners** to discover services and identify versions
- Use a **controlled credential check** against the local FTP service
- Learn to save results into Metasploit’s database (`hosts`, `services`, `workspace`)

---


## Install Metasploit 

Check:

```bash
msfconsole -v
```

If it’s missing, install:

```bash
sudo apt update
```

```bash
sudo apt install -y metasploit-framework
```

---

## Create a Safe Local Target

We’ll run a local FTP server that you control, with a known test username/password

### Install a simple FTP server library
```bash
sudo apt update
```

```bash
sudo apt install -y python3-pip
```

```bash
pip3 install --user pyftpdlib
```


### Create a folder to share
```bash
mkdir -p ~/Desktop/msf-lab/ftp-share
```

```bash
echo "hello from ftp" > ~/Desktop/msf-lab/ftp-share/hello.txt
```

### Start the local FTP server (with a test user)
We will use:
- Username: `student`
- Password: `Password123!`

Run this **in a terminal window** and keep it running:
```bash
python3 -m pyftpdlib -p 2121 -w -d ~/Desktop/msf-lab/ftp-share -u student -P 'Password123!'
```

You should see output that the server is listening on port `2121`

<img width="723" height="87" alt="image" src="https://github.com/user-attachments/assets/33d4af5f-6e40-484a-bc72-6b071e73760f" />

---

## Metasploit Console Basics

>[!NOTES]
>This should be done in another terminal, open a new window!

Start Metasploit:
```bash
msfconsole
```

### Confirm database connection inside msfconsole
Inside `msfconsole`, run:
```text
msf6 > db_status
```

If it says connected, great. If not, you can still continue (scans will work, but saving results may be limited).

## 4.2 Helpful navigation commands
Try these inside `msfconsole`:
```text
msf6 > help
msf6 > version
msf6 > show -h
msf6 > search -h
```

---

# Part 5 — Scan localhost (safe discovery)

We’ll treat `127.0.0.1` as our “target”.

## 5.1 Create a workspace (keeps your lab results separate)
```text
msf6 > workspace -a msf_beginner_lab
msf6 > workspace
```

## 5.2 Port scan with Metasploit
Use a TCP port scanner module:
```text
msf6 > search portscan tcp
```

Pick this common module:
```text
msf6 > use auxiliary/scanner/portscan/tcp
```

Show options:
```text
msf6 auxiliary(scanner/portscan/tcp) > show options
```

Set the target and ports:
```text
msf6 auxiliary(scanner/portscan/tcp) > set RHOSTS 127.0.0.1
msf6 auxiliary(scanner/portscan/tcp) > set PORTS 21,22,80,443,2121
msf6 auxiliary(scanner/portscan/tcp) > run
```

You should see `2121` open (and maybe others depending on your machine).

## 5.3 View discovered hosts/services (database)
If DB is connected:
```text
msf6 auxiliary(scanner/portscan/tcp) > hosts
msf6 auxiliary(scanner/portscan/tcp) > services
```

> If `hosts`/`services` are empty, that’s usually a DB issue. You can still proceed.

---

# Part 6 — Identify the FTP service (version scanning)

Metasploit has scanner modules that grab banners and versions.

## 6.1 Use the FTP version scanner
```text
msf6 > use auxiliary/scanner/ftp/ftp_version
msf6 auxiliary(scanner/ftp/ftp_version) > show options
```

Set target and port:
```text
msf6 auxiliary(scanner/ftp/ftp_version) > set RHOSTS 127.0.0.1
msf6 auxiliary(scanner/ftp/ftp_version) > set RPORT 2121
msf6 auxiliary(scanner/ftp/ftp_version) > run
```

You should see a banner/version line returned by the server.

Check services again (optional):
```text
msf6 auxiliary(scanner/ftp/ftp_version) > services -p 2121
```

---

# Part 7 — Controlled credential check (local FTP login scanner)

This demonstrates Metasploit’s credential modules in a **safe** way: you’re testing a service you started yourself.

## 7.1 Create a tiny password list
In a **new terminal** (not msfconsole):
```bash
cat > ~/msf-lab/passwords.txt << 'EOF'
123456
password
Password123!
letmein
EOF
```

## 7.2 Run the FTP login scanner in Metasploit
Back in `msfconsole`:
```text
msf6 > use auxiliary/scanner/ftp/ftp_login
msf6 auxiliary(scanner/ftp/ftp_login) > show options
```

Set the target and port:
```text
msf6 auxiliary(scanner/ftp/ftp_login) > set RHOSTS 127.0.0.1
msf6 auxiliary(scanner/ftp/ftp_login) > set RPORT 2121
```

Set the username:
```text
msf6 auxiliary(scanner/ftp/ftp_login) > set USERNAME student
```

Point Metasploit to the password list:
```text
msf6 auxiliary(scanner/ftp/ftp_login) > set PASS_FILE ~/msf-lab/passwords.txt
```

Run it:
```text
msf6 auxiliary(scanner/ftp/ftp_login) > run
```

Expected result: it should report a successful login for `student:Password123!`.

## 7.3 See stored credentials (if DB is connected)
```text
msf6 auxiliary(scanner/ftp/ftp_login) > creds
```

---

# Part 8 — Bonus: Quick HTTP scanning (optional, still safe)

We’ll spin up a simple local web server and scan it.

## 8.1 Start a local web server
In a **new terminal**, run:
```bash
cd ~/msf-lab
python3 -m http.server 8080
```

## 8.2 Scan HTTP title with Metasploit
In `msfconsole`:
```text
msf6 > use auxiliary/scanner/http/title
msf6 auxiliary(scanner/http/title) > set RHOSTS 127.0.0.1
msf6 auxiliary(scanner/http/title) > set RPORT 8080
msf6 auxiliary(scanner/http/title) > run
```

You should see the page title (or a response indicating the server is reachable).

---

# Part 9 — Clean up

## 9.1 Stop the local servers
- In the FTP server terminal: press **Ctrl+C**
- In the HTTP server terminal (if used): press **Ctrl+C**

## 9.2 Exit Metasploit
```text
msf6 > exit
```

## 9.3 (Optional) Stop the Metasploit DB
```bash
sudo msfdb stop
```

---

# What you learned (recap)
- How to install and start Metasploit (`msfconsole`, `msfdb`)
- How to use modules and options (`search`, `use`, `show options`, `set`, `run`)
- How scanners work (port scan + version scan)
- How a **controlled** login check works against a **local** service you own
- How to store and review results (`workspace`, `hosts`, `services`, `creds`)

---

# Troubleshooting

## “msfconsole is slow”
First run can take a bit while it initializes. Try updating packages:
```bash
sudo apt update
sudo apt install -y metasploit-framework
```

## “db_status says not connected”
Try:
```bash
sudo msfdb reinit
sudo msfdb status
```
Then reopen `msfconsole` and run `db_status` again.

## “Port 2121 already in use”
Pick another port, e.g. `2122`:
```bash
python3 -m pyftpdlib -p 2122 -w -d ~/msf-lab/ftp-share -u student -P 'Password123!'
```
Then set `RPORT 2122` in Metasploit modules.

---









---

# Finished?

[Back to Card's Main Page](/Cards/IC/External_Service_Exploitation.md)

![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Impacket

# Ubuntu VM

## In this lab we will
- Stand up a local SMB server (Samba) as our "target"
- Use Impacket to:
  - List SMB shares
  - Browse directories
  - Upload and download files
  - (Optional) query basic SMB/RPC info

## Lab topology
- Your Linux machine (attacker/workstation)
- A Samba container listening on 127.0.0.1:1445 (target)

---

## 2) Setup

### Go the the Lab Directory

```bash
cd ~/BnB/impacket
```


### Activate a virtual environment

```bash
source venv/bin/activate
```

### Verify the install

```bash
python -c "import impacket; print('Impacket version:', getattr(impacket,'__version__','(unknown)'))"
```

Also check that some scripts exist:

```bash
which smbclient.py
which smbserver.py
```

If `which` prints paths, you're good.

---
## 3) Start a local SMB "target" (Samba in Docker)

We'll run Samba on **127.0.0.1:1445** (host port 1445 -> container port 445), so it won't conflict with anything using port 445.

### Create a folder to share

```bash
mkdir -p ./target_share
echo "hello from the SMB target" > ./target_share/hello.txt
```

### Start the Samba container

Run this command exactly:

```bash
sudo docker rm -f samba-lab 2>/dev/null || true
sudo docker run -d --name samba-lab \
  -p 127.0.0.1:1445:445 \
  -v "$PWD/target_share:/mount" \
  dperson/samba \
  -p \
  -u "student;Password123!" \
  -s "share;/mount;yes;no;no;student"
```

What you just created:
- SMB user: `student`
- Password: `Password123!`
- Share name: `share`
- Files stored in: `~/impacket-lab/target_share`

### Confirm the port is listening

```bash
nc -vz 127.0.0.1 1445
```

You should see a "succeeded" message.

---
## 4) Use Impacket to list shares and browse files

### List shares on the target

```bash
smbclient.py student:Password123!@127.0.0.1 -port 1445
```

This opens an interactive SMB shell.

Inside the `smbclient.py` prompt, run:

```text
# list shares (tip: you can type `help` to see available commands)
shares
```

You should see a share named `share`.

### Enter the share and list files

Still inside the Impacket SMB shell:

```text
use share
ls
```

You should see `hello.txt`.

### Download a file from the SMB target

```text
get hello.txt
```

Exit the SMB shell:

```text
exit
```

Check that the file downloaded to your current directory:

```bash
ls -l
cat hello.txt
```

### Upload a file to the SMB target

Create a new local file:

```bash
echo "this file was uploaded using Impacket" > upload.txt
```

Reconnect:

```bash
smbclient.py student:Password123!@127.0.0.1 -port 1445
```

Inside:

```text
use share
put upload.txt
ls
exit
```

Verify on the host that the file is in the shared folder:

```bash
ls -l ./target_share
cat ./target_share/upload.txt
```

---

## 5) Run an SMB server using Impacket (reverse the roles)

So far, you used Impacket as a client to talk to Samba.

Now you'll run **Impacket's own SMB server** and connect to it.

### Create an "attacker" share folder

```bash
mkdir -p ./attacker_share
echo "hello from impacket smbserver" > ./attacker_share/from_impacket.txt
```

### Start smbserver.py on a safe port

We'll use port **2445** (so you don't need sudo).

In Terminal 1, run:

```bash
smbserver.py -port 2445 -smb2support LABSHARE ./attacker_share -username demo -password DemoPass123!
```

Leave this running.

### Connect to your Impacket SMB server

Open Terminal 2 (new terminal), and run:

```bash
source ~/impacket-lab/.venv/bin/activate
smbclient.py demo:DemoPass123!@127.0.0.1 -port 2445
```

Inside:

```text
shares
use LABSHARE
ls
get from_impacket.txt
exit
```

Back on the host:

```bash
cat from_impacket.txt
```

Stop the SMB server in Terminal 1 with:

```text
Ctrl+C
```

---

# 6) Cleanup

Stop and remove the Samba container:

```bash
sudo docker rm -f samba-lab
```

(If you started `smbserver.py`, make sure it's stopped with **Ctrl+C**)

Deactivate your Python environment:

```bash
deactivate
```

Optional: remove the lab directory:

```bash
cd ~
rm -rf ~/impacket-lab
```

---

# Troubleshooting

## "Connection refused" to 127.0.0.1:1445
- Make sure the Samba container is running:

```bash
sudo docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

You should see `samba-lab` mapped to `127.0.0.1:1445->445/tcp`.

## "Authentication failed"
- Re-check the exact username/password:
  - Username: `student`
  - Password: `Password123!`
- Restart the container (cleanup step) and start it again.

## "Command not found: smbclient.py"
- Your virtualenv might not be active:

```bash
cd ~/impacket-lab
source .venv/bin/activate
which smbclient.py
```

If it's still missing, reinstall:

```bash
python -m pip install --upgrade pip
python -m pip install --force-reinstall impacket
```

---

# What you just learned (quick recap)
- Impacket is a collection of Python tools that speak real network protocols.
- You used it as an SMB client (`smbclient.py`) for enumeration + file operations.
- You also used it as an SMB server (`smbserver.py`) to host a share.



---

# Finished?

[Back to Keberoasting's Main Page](/Cards/PE/Kerberoasting.md)

[Back to Local_Privilege_Escalation's Main Page](/Cards/PE/Local_Privilege_Escalation.md)

![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Shodan

## What you’ll do in this lab
- Install the **Shodan CLI**
- Connect the CLI using your **API key**
- Run basic searches and filters (country, org, ports)
- Inspect a host (services, banners)
- Export results to files (download + parse)
- Check your own public IP (and understand common “no results” cases)

---

## Create a Shodan account
Create an account on Shodan [HERE](https://account.shodan.io/register)

<img width="734" height="540" alt="image" src="https://github.com/user-attachments/assets/275aef2c-df81-4661-a81b-8faa3808df44" />

---

## First search: “What is exposed on the internet?”
Shodan searches **service banners** (the text a service reveals when scanned), not just open ports.

### Count results (fast)
```bash
shodan count "apache"
```

### Get a few results (read-only)
Show the first 5 results and only a few fields:

```bash
shodan search --limit 5 --fields ip_str,port,org,hostnames "apache"
```

**What to look for**
- `ip_str`: the IP address
- `port`: the exposed service port
- `org`: the network owner (sometimes empty)
- `hostnames`: DNS names (sometimes empty)

---

## Use simple filters (country, port, product)
Shodan queries support powerful filters. Here are beginner-friendly ones.

### Filter by country (example: Romania = RO)
```bash
shodan count "nginx country:RO"
shodan search --limit 5 --fields ip_str,port,org,hostnames "nginx country:RO"
```

### Filter by port
```bash
shodan count "port:22"
shodan search --limit 5 --fields ip_str,port,org,hostnames "port:22"
```

### Filter by product (banner-identified software)
```bash
shodan count "product:OpenSSH"
shodan search --limit 5 --fields ip_str,port,org,hostnames "product:OpenSSH"
```

>[!Notes]
> “product” depends on banner detection. Not every service reports product info.

---

## ummarize results with facets (top countries, orgs, ports)
Facets give you quick “top N” summaries without pulling lots of results.

### Top countries for OpenSSH
```bash
shodan search --limit 0 --facets country "product:OpenSSH"
```

### Top organizations for OpenSSH
```bash
shodan search --limit 0 --facets org "product:OpenSSH"
```

### Top ports for “nginx”
```bash
shodan search --limit 0 --facets port "nginx"
```

---

## Host investigation (inspect one IP)
Pick **one IP** from your previous search output (Step 4 or 5) and set it as a variable:

```bash
IP="PUT_AN_IP_HERE"
```

Now run:

```bash
shodan host "$IP"
```

**What you’re seeing**
- Open ports discovered by Shodan
- Service banners (often includes server type, certificates, page titles, etc.)
- Sometimes tags like “vpn”, “compromised” (depends on Shodan data)

### Save host data as JSON (for later review)
```bash
shodan host "$IP" --history --format json > host.json
ls -lh host.json
```

### Read the JSON safely with jq
Show the top-level keys:

```bash
jq 'keys' host.json
```

Show ports:

```bash
jq '.ports' host.json
```

Show a compact list of services Shodan saw:

```bash
jq -r '.data[] | "\(.port) \(.transport) \(.product // "unknown") \(.version // "")"' host.json | head -n 20
```

---

## Export and parse search results (download -> parse)
This is useful for reporting or later analysis.

### Download a dataset (example query)
Pick a simple query, for example:

```bash
QUERY='nginx country:RO'
```

Download:

```bash
shodan download my_nginx_ro "$QUERY"
```

You should get a `.json.gz` file. Confirm:

```bash
ls -lh my_nginx_ro.json.gz
```

### Parse the downloaded file into a clean table
```bash
shodan parse --fields ip_str,port,org,hostnames my_nginx_ro.json.gz | head -n 20
```

### Export to CSV (easy for Excel)
```bash
shodan parse --fields ip_str,port,org,hostnames --separator , my_nginx_ro.json.gz > my_nginx_ro.csv
ls -lh my_nginx_ro.csv
```

---

## Check your own public IP (and why you might see “no data”)
### Get your public IP
```bash
MYIP="$(curl -s ifconfig.me)"
echo "My public IP is: $MYIP"
```

### 9.2 Try Shodan host lookup
```bash
shodan host "$MYIP"
```

If you get **no results**, that’s common! Reasons include:
- Your router/NAT hides your internal devices
- Your ISP blocks inbound ports
- Shodan hasn’t scanned your IP recently
- Your IP changed recently

### Use Shodan InternetDB (free, no API key required)
InternetDB is a simple endpoint that often returns basic exposure info:

```bash
curl -s "https://internetdb.shodan.io/$MYIP" | jq
```

**What to look for**
- `ports`: which ports are exposed
- `hostnames`: known DNS
- `cpes`: product identifiers (if any)
- `vulns`: known CVE mappings (if any)

> If this returns empty ports, that can still be totally normal and means you’re not publicly exposed (good!).

---

## Mini challenge
Try answering these using **only Shodan searches**:

1) Find how many hosts expose **RDP (3389)** in your country:
```bash
shodan count "port:3389 country:RO"
```

2) Find a common port for **MongoDB** exposures:
```bash
shodan search --limit 0 --facets port "product:MongoDB"
```

3) Find the top countries for **FTP**:
```bash
shodan search --limit 0 --facets country "port:21"
```

---

## Cleanup
Remove downloaded datasets:

```bash
rm -f my_nginx_ro.json.gz my_nginx_ro.csv host.json
```

---

## Quick reference: useful beginner queries
Use these as “copy/paste starters”:

```bash
shodan count "port:22"
shodan count "port:3389"
shodan count "http.title:login"
shodan search --limit 5 --fields ip_str,port,org,hostnames "product:OpenSSH"
shodan search --limit 0 --facets country,org,port "nginx"
```











---

# Finished?

[Back to Card's Main Page](/Cards/IC/External_Service_Exploitation.md)

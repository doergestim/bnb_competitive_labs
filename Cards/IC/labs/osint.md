![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# OSINT

# Ubuntu VM

## In this lab we will
- Harvest emails, subdomains, and IPs from public sources using **theHarvester**
- Hunt usernames across dozens of social platforms using **Sherlock**
- Extract DNS and domain ownership data using **WHOIS** and **dig**
- Understand what an attacker (or analyst) can find without touching a target

---

## Tool 1 - theHarvester

theHarvester collects emails, subdomains, IPs, and URLs from public sources like Google, Bing, and various DNS databases.

### What sources does theHarvester use?

List all available data sources:

```bash
theHarvester -d hackthissite.org -b all -l 10
```

You will see sources like `crtsh`, `dnsdumpster`, `virustotal`, and more. Each one is a different public service being queried. For many, you will need API keys for full functionality

---

### Harvesting a domain - subdomains

Let's run a basic subdomain harvest against `hackthissite.org` using the `crtsh` source (Certificate Transparency logs - completely passive):

```bash
theHarvester -d hackthissite.org -b crtsh
```

You will see a list of subdomains like:

<img width="688" height="841" alt="image" src="https://github.com/user-attachments/assets/0eb5dce1-9414-4302-940e-bbf7fa0b4c61" />



These are pulled from SSL certificate logs

---

### Harvesting historical URLs

The Wayback Machine and Common Crawl are public archives that have been indexing the internet for decades. theHarvester can query them to find old URLs, forgotten endpoints, and pages that may no longer be linked anywhere:

```bash
theHarvester -d hackthissite.org -b waybackarchive,commoncrawl -l 100
```

You will see URLs or subdomains like:

<img width="952" height="726" alt="image" src="https://github.com/user-attachments/assets/c69ea3cf-c399-42c2-8ce7-387ae8ae60dd" />


Why does this matter? Old URLs can reveal forgotten admin panels, legacy login pages, deprecated API endpoints, or backup files that were never removed. Attackers look for these because they are often unpatched and unmonitored. An analyst can use this to build a list of endpoints to review during a security assessment.

---

## Tool 2 - Sherlock

Sherlock searches over 300 social platforms and websites for a given username. It tells you where that username exists.

### Search for a username

Let's search for the username `johndoe`:

```bash
sherlock johndoe
```

Sherlock will check hundreds of sites and print results like:

<img width="1913" height="971" alt="image" src="https://github.com/user-attachments/assets/eeff70e9-796a-4ada-b012-bc6b5b3142e9" />


A `[+]` means the profile exists. A `[-]` means it does not.

---

### Search for multiple usernames at once

```bash
sherlock alice bob charlie
```

Useful when a target uses multiple aliases - you check all of them in one command.

---

### Why does this matter?

Imagine an attacker finds a username from a data breach. With Sherlock they can:
- Find the target's LinkedIn -> job history, colleagues
- Find their GitHub -> leaked API keys, internal repos
- Find their Reddit -> personal details, location clues

All in under a minute.

---

## Tool 3 - WHOIS + dig

These tools are built into every Linux system and give you domain ownership, registrar info, DNS records, and server IPs.

### WHOIS - Who owns a domain?

```bash
whois github.com
```

Look for:
- **Registrant** - who registered the domain
- **Registrar** - where it was registered (GoDaddy, Namecheap, etc.)
- **Creation Date** - how old the domain is (new domains are suspicious)
- **Name Servers** - where DNS is hosted

```bash
whois hackthissite.org | grep -E "Registrar|Creation|Name Server|Registrant"
```

This filters the output to only the most relevant lines.

---

### WHOIS on an IP address

You can also run WHOIS on an IP to find who owns that IP block:

```bash
whois 8.8.8.8
```

You will see it belongs to Google. Try:

```bash
whois 1.1.1.1
```

This belongs to Cloudflare. Knowing who owns an IP helps during incident response - is the traffic going to AWS, a VPS provider, or a residential ISP?

---

### dig - DNS record enumeration

`dig` queries DNS servers directly.

Get the A record (IP address) of a domain:

```bash
dig hackthissite.org A
```

Get all mail servers (MX records):

```bash
dig hackthissite.org MX
```

Get name servers:

```bash
dig hackthissite.org NS
```

Get the TXT records (often contains SPF, DKIM, verification tokens):

```bash
dig hackthissite.org TXT
```

TXT records can leak internal information - verify tokens, third-party integrations, sometimes internal hostnames.

---

### Query a specific DNS server

Instead of using your ISP's DNS, query Google's directly:

```bash
dig @8.8.8.8 hackthissite.org A
```

This is useful when you want to bypass local DNS caching or test DNS propagation.

---

### Reverse DNS lookup

Given an IP, find what hostname it resolves to:

```bash
dig -x 140.82.121.4
```

This is the reverse of a normal DNS query. Useful for identifying what a suspicious IP belongs to during log analysis.

---

### Combine it all - a full manual recon run

Here is a mini recon workflow you would run against a target domain. Use `hackthissite.org` as the example:

```bash
echo "=== WHOIS ===" && whois hackthissite.org | grep -E "Registrar|Creation|Name Server"
echo "=== A RECORD ===" && dig hackthissite.org A +short
echo "=== MX RECORDS ===" && dig hackthissite.org MX +short
echo "=== TXT RECORDS ===" && dig hackthissite.org TXT +short
echo "=== NAME SERVERS ===" && dig hackthissite.org NS +short
```

Run this as a one-liner. You now have a quick DNS profile of the target in seconds.

---

## Putting it all together

A real OSINT workflow for a target company might look like this:

**Step 1 - Find the domain and subdomains:**
```bash
theHarvester -d hackthissite.org -b crtsh -f /tmp/hts_recon
```

**Step 2 - Pull emails and any leaked data:**
```bash
theHarvester -d hackthissite.org -b google,bing -l 200 -f /tmp/hts_emails
```

**Step 3 - Get DNS and ownership info:**
```bash
whois hackthissite.org
dig hackthissite.org A MX TXT NS
```

**Step 4 - Search for public usernames tied to the platform:**
```bash
sherlock hts --print-found
```

Each step builds a picture. Subdomains -> attack surface. Emails -> phishing targets. DNS records -> infrastructure layout. Usernames -> personal accounts that may expose more data.

---

## What to think about

- Everything in this lab used **publicly available data** - no exploitation occurred
- OSINT is the first phase of almost every real-world attack
- As a defender, run these tools against **your own organization** periodically to see what attackers see
- If theHarvester finds internal-looking subdomains, leaked emails, or dev/staging servers exposed to the internet - that is a finding worth reporting










---

# Finished?

[Back to Card's Main Page](/Cards/IC/Social_Engineering.md)

![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Gato-X

# Ubuntu VM

## Background - What is Gato-X?

Gato-X is an open-source GitHub Actions security auditing tool. It was built to help security teams find misconfigurations in GitHub Actions workflows before attackers do.

It looks for things like:

- **Pwn Requests** - workflows using `pull_request_target` that check out and run untrusted code from a fork, allowing an outsider to steal secrets or run arbitrary commands in a privileged context
- **Self-hosted runner abuse** - self-hosted runners that can be hijacked
- **Secret exposure** - secrets accidentally printed in workflow logs

Real-world impact: attackers who find these misconfigurations in public repos can execute code in your CI/CD environment and steal tokens, API keys, and credentials.

---

## Create a GitHub Personal Access Token (PAT)

Gato-X needs a token to talk to the GitHub API. We will create one with the minimum required scopes.

1. Go to [https://github.com/settings/tokens](https://github.com/settings/tokens)
2. Click **"Generate new token"** -> **"Generate new token (classic)"**

<img width="957" height="401" alt="2026-04-14_14-22" src="https://github.com/user-attachments/assets/890f336c-e10f-40f8-b44b-0984d5784d54" />


3. Give it a name like `gato-x-lab`
4. Set expiration to **7 days** (this is just for the lab)

5. Under **Scopes**, check:
   - `repo` (full repo access)
   - `read:org`
   - `workflow`

<img width="889" height="784" alt="2026-04-14_14-26" src="https://github.com/user-attachments/assets/8b0fe1dd-d8aa-45c7-8a0f-180107ec2e9e" />


6. Click **"Generate token"**
7. **Copy the token immediately** - you will not see it again

<img width="490" height="101" alt="2026-04-14_14-27" src="https://github.com/user-attachments/assets/2bd86f0a-14ff-4f24-bafa-5ba6f2c76a44" />


Save your token somewhere safe for this session:

```bash
export GITHUB_TOKEN="ghp_YourTokenHere"
```

> **Security note:** Never commit a token to a repo. Never share it. Revoke it after the lab at [https://github.com/settings/tokens](https://github.com/settings/tokens).

---

## Create a Vulnerable Test Repository

We will create a GitHub repo with a **deliberately misconfigured** workflow. This is the exact type of vulnerability Gato-X is designed to find.

### Create the repo on GitHub

1. Go to [https://github.com/new](https://github.com/new)
2. Name it: `gato-x-vulnerable-test`
3. Set it to **Public** (Gato-X's enumerate works on public repos; this is your own test repo)
4. Check **"Add a README file"**
5. Click **"Create repository"**

<img width="1019" height="868" alt="2026-04-14_14-28" src="https://github.com/user-attachments/assets/f7205f23-e3ff-413e-ae9f-a012fb01dc46" />


### Step 2 - Clone it locally

```bash
git clone https://github.com/YOUR_USERNAME/gato-x-vulnerable-test.git
```

```bash
cd gato-x-vulnerable-test
```

Replace `YOUR_USERNAME` with your actual GitHub username.

### Create the vulnerable workflow

Create the workflows directory:

```bash
mkdir -p .github/workflows
```

Create the vulnerable workflow file:

```bash
nano .github/workflows/vulnerable.yml
```

Paste the following content:

```yaml
name: Vulnerable CI Workflow

on:
  pull_request_target:
    types: [opened, synchronize]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout PR code
        uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Run build script from PR
        run: |
          bash ./build.sh
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X` in nano).

> **Why is this vulnerable?**
> `pull_request_target` runs in the context of the **base repo** (with access to its secrets), but this workflow checks out the **PR submitter's code** (`head.sha`) and runs it directly. An attacker could fork your repo, submit a PR with a malicious `build.sh`, and their code runs with access to your repo's secrets.

### Push it to GitHub

```bash
git add .github/workflows/vulnerable.yml
git commit -m "Add vulnerable workflow for lab"
git push origin main
```

---

## Run Gato-X Enumeration

Now we use Gato-X to scan our own repository and find the vulnerability we just introduced.

### Enumerate your own user account

This scans all repos under your account:

```bash
gato-x enumerate --token $GITHUB_TOKEN --target YOUR_USERNAME
```

Replace `YOUR_USERNAME` with your GitHub username.

Watch the output. Gato-X will:
1. Authenticate using your token
2. Pull a list of your repositories
3. Check each repository's workflow files for misconfigurations
4. Report any findings

### Enumerate a specific repository

You can also target just one repo directly:

```bash
gato-x enumerate --token $GITHUB_TOKEN --target YOUR_USERNAME/gato-x-vulnerable-test
```

---

## Read the Output

You should see output that looks something like this:

```
[!] Potentially vulnerable workflow detected!
    Repository : YOUR_USERNAME/gato-x-vulnerable-test
    Workflow   : vulnerable.yml
    Trigger    : pull_request_target
    Issue      : Workflow checks out PR head code and executes it
    Severity   : HIGH - attacker-controlled code runs in privileged context
```

Let's break down what this means as an analyst:

| Field | What It Means |
|---|---|
| **Repository** | The repo that has the problem |
| **Workflow** | The specific `.yml` file that is misconfigured |
| **Trigger** | `pull_request_target` - runs with base repo privileges |
| **Issue** | PR head code is being executed directly |
| **Severity** | HIGH - an outside attacker can run code and access secrets |

---

## Save the Output to a File

In a real engagement you always save your output. Use the output flag:

```bash
gato-x enumerate --token $GITHUB_TOKEN --target YOUR_USERNAME --output results.txt
```

View the saved results:

```bash
cat results.txt
```

---

## Search for Vulnerable Public Repos

Gato-X can also search GitHub for **publicly known** vulnerable workflow patterns. This is a **read-only search** - we are not exploiting anything, just identifying

```bash
gato-x search --token $GITHUB_TOKEN --query "pull_request_target"
```

---

## Fix the Vulnerability

Understanding the fix is just as important as finding the problem

Open the workflow file:

```bash
nano .github/workflows/vulnerable.yml
```

Replace it with the **safe version**:

```yaml
name: Safe CI Workflow

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - name: Checkout PR code
        uses: actions/checkout@v3

      - name: Run build script
        run: |
          bash ./build.sh
```

Key changes made:
- Changed `pull_request_target` -> `pull_request` (runs in the fork's context, not the base repo's privileged context)
- Added `permissions: contents: read` to restrict what the job can do
- Removed the explicit `ref: ${{ github.event.pull_request.head.sha }}` checkout - the default `pull_request` trigger checks out a safe merge commit

Push the fix:

```bash
git add .github/workflows/vulnerable.yml
git commit -m "Fix: replace pull_request_target with pull_request"
git push origin main
```

### Re-scan to confirm the fix

```bash
gato-x enumerate --token $GITHUB_TOKEN --target YOUR_USERNAME/gato-x-vulnerable-test
```

Gato-X should no longer report the vulnerability.









---

# Finished?

[Back to Card's Main Page](/Cards/IC/Trusted_Relationship.md)

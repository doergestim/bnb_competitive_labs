# For the Ubuntu VM

## The objective of this lab is to use Scout Suite to audit an AWS cloud environment and identify critical security misconfigurations related to Identity Access Management (IAM), Cloud Storage (S3), and Virtual Machines (EC2).

If you want to learn a bit about this tool check the [Scout Suite Documentation](https://www.google.com/search?q=/courseFiles/tools/ScoutSuite.md)

### Phase 1: Account Creation 

- Go to the [AWS Free Tier web page](https://aws.amazon.com/free/) and click on **"Create a Free Account"**.
  
<img width="1417" height="479" alt="image" src="https://github.com/user-attachments/assets/c5b52069-a4b6-48d1-87c1-377b4fb7a2bc" />

- Fill the email and account name fields:
  
<img width="856" height="784" alt="image" src="https://github.com/user-attachments/assets/3dc88976-f9aa-41b4-84aa-de44eb76c893" />

- After validating your email you will be asked to set the root password. Fill the two fields:
  
<img width="757" height="674" alt="image" src="https://github.com/user-attachments/assets/03a40be9-5e6e-4a88-9727-1348f6da57f3" />

- To demonstrate ScoutSuite usage the Paid plan is not required. Choose the **Free Plan**:

<img width="870" height="661" alt="image" src="https://github.com/user-attachments/assets/7e11a906-1f39-4a8c-84c1-f3d9d245aea4" />

>[!IMPORTANT]
> Amazon requires a valid physical address, a phone number for identity verification and a valid credit/debit card. This is a mandatory step for all cloud providers.
> <br></br>**What to do:** Fill in your contact information. Since this is a personal **"Free Tier"** account, your billing address should match the one associated with your credit/debit card. 

<img width="647" height="788" alt="image" src="https://github.com/user-attachments/assets/d329d720-1cce-4917-b914-374861da8906" />

- Fill the Billing information. Amazon will only hold **$1** for 3-5 days to verify your account.
  
<img width="734" height="955" alt="image" src="https://github.com/user-attachments/assets/1820f380-21bb-411f-9d9c-7c2fa0a047f7" />

- Verify your phone number: 

<img width="625" height="424" alt="image" src="https://github.com/user-attachments/assets/d9a41ce2-ddae-4a7b-8946-0ceb9c8d0898" />

- You should now have access to the **Amazon Web Services**. This will grant you : 
  - USD $100 in credits 
  - access to 30+ AWS services
    
>[!TIP] 
> Scout Suite is designed to perform security audits on AWS. It leverages APIs to scan for misconfigurations-such as open security groups, unencrypted S3 buckets and overly permissive IAM roles.

-----
### Phase 2: Creating the Target Environment (AWS Console)

Note: Before using our tools, we need to act as an administrator and intentionally misconfigure a few services in the AWS Web Console to give our scanner something to find.

  - *1. Create a Vulnerable EC2 Instance:*

      - Go to EC2 -\> Launch Instance. Choose the t2.micro (Free Tier).
      - Under *Network Settings*, create a new Security Group named Vulnerable-SG.
      - Add an Inbound Rule: Type SSH, Port 22, Source Anywhere (0.0.0.0/0). Launch the instance.

  - *2. Create an Exposed S3 Bucket:*

      - Go to S3 -\> Create Bucket. Name it uniquely (e.g., sec-lab-exposed-data-[yourname]).
      - *Uncheck* "Block all public access" and acknowledge the warning.
      - Once created, upload a fake text file (e.g., passwords.txt).

  - *3. Create the Audit User & Generate Keys (IAM):*

      - Go to IAM -\> Users -\> Create User. Name it scout-auditor.
      - Attach policies directly: Search for and check ReadOnlyAccess and SecurityAudit.
      - Once created, click on the user -\> Security credentials -\> *Create access key*. Select "Command Line Interface (CLI)".
      - *SAVE the Access Key ID and Secret Access Key\!* You will need them in the terminal.

\<img width="800" height="300" alt="image" src="[PLACEHOLDER\_FOR\_IAM\_KEYS\_SCREENSHOT]" /\>

-----

### Phase 3: Local VM Setup & Virtual Environment

Now we move to the Ubuntu terminal to set up our isolated Python environment for Scout Suite.

  - First, create a new directory for the lab and move into it.

<!-- end list -->

```bash
mkdir -p /home/ubuntu/SOC_Analyst_Labs/ScoutSuite_Lab
cd /home/ubuntu/SOC_Analyst_Labs/ScoutSuite_Lab
```

  - Create a Python Virtual Environment named scout-env. This acts as an isolated container for the tool's dependencies.

<!-- end list -->

```bash
python3 -m venv scout-env
```

  - Activate the virtual environment. You should see (scout-env) appear at the beginning of your terminal prompt.

<!-- end list -->

```bash
source scout-env/bin/activate
```

\<img width="600" height="100" alt="image" src="[PLACEHOLDER\_FOR\_VENV\_ACTIVATION\_SCREENSHOT]" /\>

  - With the environment active, install Scout Suite using pip.

<!-- end list -->

```bash
pip install scoutsuite
```

-----

### Phase 4: Authentication and Execution

  - We must authenticate our local VM with the AWS Cloud using the credentials generated in Phase 2. Run the configuration command and input your Access Key ID, Secret Access Key, and set the default region (e.g., us-east-1).

<!-- end list -->

```bash
aws configure
```

\<img width="600" height="200" alt="image" src="[PLACEHOLDER\_FOR\_AWS\_CONFIGURE\_SCREENSHOT]" /\>

  - Before launching the scanner, let's verify that our API connection works and we are logged in as scout-auditor.

<!-- end list -->

```bash
aws sts get-caller-identity
```

\<img width="600" height="150" alt="image" src="[PLACEHOLDER\_FOR\_CALLER\_IDENTITY\_SCREENSHOT]" /\>

  - It's time to run the security audit. We will specify aws as our target cloud provider. Scout Suite will now fetch configuration data across all active services.

<!-- end list -->

```bash
scout aws
```

\<img width="921" height="500" alt="image" src="[PLACEHOLDER\_FOR\_SCOUT\_AWS\_EXECUTION\_SCREENSHOT]" /\>

  - Once the scan is complete, an interactive HTML report is generated in the scoutsuite-report directory. Verify its creation.

<!-- end list -->

ls -lh scoutsuite-report
```

  - Finally, open the generated .html file in your web browser.
  - *Your Task:* Analyze the dashboard and locate the intentionally vulnerable EC2 instance (Port 22 open) and the publicly exposed S3 Bucket.

\<img width="921" height="600" alt="image" src="[PLACEHOLDER\_FOR\_HTML\_DASHBOARD\_SCREENSHOT]" /\>

## 🔍 Phase 1: IAM Lab Setup — Creating the Misconfigured Environment

In this first phase, I set up the foundation for the privilege escalation simulation. The goal is to create two IAM users:

- `AdminUser`: Full access (for setup only)  
- `LimitedUser`: A deliberately restricted user with a custom policy that looks safe — but has hidden privilege escalation potential

---

### ✅ Step 1: Login to AWS Console as Root

I logged in using my root credentials since it's the only account I currently manage. From the AWS console:

- Searched for **IAM**
- Clicked **Users > Create User**

---

### ✅ Step 2: Create the "AdminUser" IAM Account

This user simulates a cloud admin. I gave it full admin access temporarily to configure the lab:

- Username: `AdminUser`  
- Access type: **Programmatic access + AWS Console access**  
- Set a custom password and ticked "User must reset password"  
- Attached policy: **AdministratorAccess**

✅ I saved the credentials, logged in, and confirmed access.

📸 Screenshot: `AdminUser` created  
<img width="960" height="505" alt="image" src="https://github.com/user-attachments/assets/8abe63dc-659a-456d-bcd3-9caacdae8d2e" />

---

### ✅ Step 3: Create the "LimitedUser" IAM Account

Next, I created a second user with limited permissions — but **intentionally left a loophole** for privilege escalation:

- Username: `LimitedUser`  
- Access type: **Programmatic access + AWS Console access**  
- Custom password (tested it ✅)  
- **Attached a custom inline policy** named `PassRoleExploitPolicy`

Here’s the exact JSON I used:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:PassRole",
        "iam:AttachUserPolicy"
      ],
      "Resource": "*"
    }
  ]
}
```

📸 Screenshot: `LimitedUser` inline policy  
<img width="960" height="505" alt="image" src="https://github.com/user-attachments/assets/9035194d-23b7-4846-a1be-e9cd939d8d13" />

---

## 🚨 Phase 2: Privilege Escalation — Exploiting IAM Misconfigurations

Now that both IAM users are created, I simulated a privilege escalation attack from the perspective of `LimitedUser`.  
The goal is to show how a seemingly harmless permission like `iam:PassRole` can lead to full admin access if not restricted properly.

---

### ✅ Step 1: Login as LimitedUser (Attacker)

Using the `LimitedUser` credentials, I connected via CLI:

```bash
aws configure
```

I provided:

- **Access Key ID**  
- **Secret Access Key**  
- **Region** (e.g., `us-east-1`)  
- **Output Format** (left blank)

Then verified identity:

```bash
aws sts get-caller-identity
```

✅ Output confirmed I was authenticated as `LimitedUser`.

---

### ✅ Step 2: Privilege Escalation via Policy Attachment

Since `LimitedUser` had the power to attach policies, I abused it to gain full admin rights:

```bash
aws iam attach-user-policy \
  --user-name LimitedUser \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

Then confirmed access:

```bash
aws iam list-users
```

✅ It worked — `LimitedUser` now had full admin power. 🔓

📸 Screenshot: `LimitedUser` escalation in action  
<img width="1920" height="909" alt="image" src="https://github.com/user-attachments/assets/b0f3daef-151d-40ae-9826-b92ba57a204d" />

---

This simulates a **real-world cloud misconfiguration** where overly permissive IAM policies allow privilege escalation — even without initial admin access.

---

## 🔍 Phase 3: Detection & Analysis — Catching the Privilege Escalation in CloudTrail & Splunk

After the escalation, I focused on **detecting** it via **CloudTrail logs** and **Splunk analysis**.

---

### ✅ Step 1: Check CloudTrail for Suspicious Activity

I navigated to **CloudTrail > Event History** and filtered for:

- **Username**: `LimitedUser`  
- **Event source**: `iam.amazonaws.com`  
- **Event name**: `AttachUserPolicy`

This revealed the exact moment `LimitedUser` attached `AdministratorAccess` to itself.

📸 Screenshot: CloudTrail showing `AttachUserPolicy`  
<img width="1920" height="1010" alt="image" src="https://github.com/user-attachments/assets/27d0a1b7-ebce-4994-b19b-8f474a5a3aa5" />

---

### ✅ Step 2: Review Event JSON for Forensic Insight

I opened the event and reviewed the full JSON fields:

- `eventTime`, `sourceIPAddress`, `userIdentity`, `requestParameters`

This helps during forensic investigations and timeline correlation.

📸 Screenshot: CloudTrail event details  
<img width="1920" height="1010" alt="image" src="https://github.com/user-attachments/assets/6938ab1d-575a-4282-bb53-50470c21e35f" />

---

### ✅ Step 3: Search in Splunk

Since CloudTrail logs were forwarding to Splunk, I ran this SPL query:

```spl
index=aws source="*:cloudtrail" eventName="AttachUserPolicy"
```

✅ Splunk returned the privilege escalation event from `LimitedUser`.

📸 Screenshot: Splunk showing the event  
*Insert screenshot if available*

---

## 🛡️ Phase 4: Remediation — Fixing the Misconfiguration

After successfully simulating and detecting the escalation, I took steps to fully remediate the insecure IAM policy setup.

---

### ✅ Step 1: Detach Dangerous Policy from `LimitedUser`

First, I removed the `AdministratorAccess` policy:

```bash
aws iam detach-user-policy \
  --user-name LimitedUser \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

Then verified access was revoked:

```bash
aws iam list-users
```

🔒 It returned `AccessDenied` — privilege escalation was successfully revoked.

📸 Screenshot: Detach confirmation  
<img width="1920" height="909" alt="VirtualBox_Kali Linux_13_07_2025_01_29_19" src="https://github.com/user-attachments/assets/a179e298-d3cd-478c-ae9b-d19c4b87e3f4" />

---

### ✅ Step 2: Delete the Inline `PassRoleExploitPolicy`

Since `LimitedUser` could no longer delete it via CLI, I switched to **AWS Console**:

- Logged in as `AdminUser`
- Navigated to **IAM > Users > LimitedUser > Permissions**
- Manually deleted the `PassRoleExploitPolicy` inline policy

✅ This removed `iam:PassRole` and `iam:AttachUserPolicy` completely.

📸 Screenshot: Inline policy removed via Console  
<img width="1920" height="1010" alt="image" src="https://github.com/user-attachments/assets/5f6095e2-06a1-4399-9c4a-f252061d7d46" />

---

### ✅ Step 3: Replace with Least Privilege Policy

I created a new policy with **minimal permissions**, following best practices:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "*"
    }
  ]
}
```

📸 Screenshot: Least privilege policy JSON  
<img width="1920" height="1010" alt="image" src="https://github.com/user-attachments/assets/0d1d65c2-4aad-4d69-89ab-8818e2905126" />

Then I applied it via CLI:

```bash
aws iam put-user-policy \
  --user-name LimitedUser \
  --policy-name LeastPrivilegePolicy \
  --policy-document file://least-privilege-policy.json
```

📸 Screenshot: Policy applied successfully  
<img width="1920" height="1010" alt="image" src="https://github.com/user-attachments/assets/e93fc6af-9bca-406c-bd94-d17f5d9d3505" />

---

### 🧠 Final Thoughts

This full IAM escalation lab proves the importance of:

- 🔐 **Principle of Least Privilege**  
- 👁️‍🗨️ **CloudTrail Monitoring + Alerting**  
- 🔍 **Regular IAM Policy Audits**

Even non-admin users can gain full control if IAM policies aren’t locked down.  
In the cloud, **what you don’t see can hurt you** — this lab made that real.


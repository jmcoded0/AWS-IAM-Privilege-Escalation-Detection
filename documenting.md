
## 🔍 Phase 1: IAM Lab Setup — Creating the Misconfigured Environment

In this first phase, I set up the foundation for the privilege escalation simulation. The goal is to create two IAM users:

- `AdminUser`: Full access (for setup only)
- `LimitedUser`: A deliberately restricted user with a custom policy that looks safe — but has hidden privilege escalation potential


### ✅ Step 1: Login to AWS Console as Root
I logged in using my root credentials since it's the only account I currently manage. From the AWS console:

- I searched for **IAM** in the service search bar
- Clicked **Users** > **Create User**

---

### ✅ Step 2: Create the "AdminUser" IAM Account
This user will help me simulate the role of a cloud admin. I gave it full admin access temporarily to help configure the lab:

- Username: `AdminUser`
- Access type: **Programmatic access + AWS Console access**
- Set a custom password and tick "User must reset password"
- Attached policy: **AdministratorAccess**

I saved the credentials and logged in to test it — it worked ✅
(screenshots/01_adminuser_created.png) <img width="960" height="505" alt="image" src="https://github.com/user-attachments/assets/8abe63dc-659a-456d-bcd3-9caacdae8d2e" />

---

### ✅ Step 3: Create the "LimitedUser" IAM Account
Next, I created a second user with limited permissions but **intentionally left a loophole** for privilege escalation:

- Username: `LimitedUser`
- Access type: **Programmatic access + AWS Console access**
- Custom password (tested it too)
- **Attached a custom inline policy** — I named it `PassRoleExploitPolicy`

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
(screenshots/02_limiteduser_policy_json.png) <img width="960" height="505" alt="image" src="https://github.com/user-attachments/assets/9035194d-23b7-4846-a1be-e9cd939d8d13" />

## 🚨 Phase 2: Privilege Escalation via PassRole + AttachUserPolicy

In this phase, I simulated a **privilege escalation attack** by taking advantage of an overly permissive IAM policy. The `LimitedUser` was only supposed to have restricted access, but the attached inline policy (`PassRoleExploitPolicy`) allowed dangerous actions like:

- `iam:PassRole` — lets a user assign roles with higher privileges  
- `iam:AttachUserPolicy` — lets a user attach **any** policy, including `AdministratorAccess`, to themselves

---

### ✅ Step 1: Login as LimitedUser

I logged in to the AWS console using the `LimitedUser` credentials.

To verify my identity and current permissions, I ran the following AWS CLI command:

\`\`\`bash
aws sts get-caller-identity
\`\`\`

✅ This confirmed I was logged in as `LimitedUser`.

📸 Screenshot:  
- ![](screenshots/03_limiteduser_loggedin_identity.png)

---

### ✅ Step 2: Exploit AttachUserPolicy to Gain Admin Rights

Even though `LimitedUser` looked restricted, it had permission to attach *any* policy to *any* user — including itself.

I ran this command in CloudShell while logged in as `LimitedUser`:

\`\`\`bash
aws iam attach-user-policy \
  --user-name LimitedUser \
  --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
\`\`\`

✅ No error was returned — this means the attack was successful.

📸 Screenshot:  
- ![](screenshots/04_attach_admin_policy_success.png)

---

### ✅ Step 3: Confirm Privilege Escalation

After attaching the `AdministratorAccess` policy to myself (`LimitedUser`), I verified the escalation by trying a privileged action.

Example command:

\`\`\`bash
aws iam list-users
\`\`\`

✅ The command succeeded, proving I now had elevated permissions as a regular user.

📸 Screenshot:  
- ![](screenshots/05_privileged_action_success.png)

---

### 🧠 Summary

This simulation shows how **subtle IAM misconfigurations** can lead to **full account compromise**. Even a harmless-looking policy like:

\`\`\`json
{
  "Action": ["iam:PassRole", "iam:AttachUserPolicy"],
  "Effect": "Allow",
  "Resource": "*"
}
\`\`\`

…can be abused by attackers if not properly scoped and monitored.

---

### 📸 Screenshots Recap (Phase 2)

- `03_limiteduser_loggedin_identity.png`  
- `04_attach_admin_policy_success.png`  
- `05_privileged_action_success.png`


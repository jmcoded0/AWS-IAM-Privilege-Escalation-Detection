# 🔐 AWS IAM Privilege Escalation Detection

This project simulates a real-world AWS IAM privilege escalation scenario. I deliberately created a misconfigured IAM environment, escalated privileges from a low-level user, and detected the attack using AWS native tools like **CloudTrail**, **GuardDuty**, and **Lambda**. The goal: understand IAM policy weaknesses, simulate attacker behavior, and build a basic detection pipeline.

---

---

## 🧠 Objective

- Simulate IAM misconfiguration that enables **privilege escalation**  
- Use AWS CLI to perform the attack (simulating an adversary)  
- Detect malicious behavior with **CloudTrail** and **GuardDuty**  
- Forward detection alerts using **Lambda** functions  
- Document the attack path, detection events, and response  

---

## 🛠️ Tools & Services Used

| Service     | Purpose                              |
|-------------|------------------------------------|
| **IAM**         | Create limited & misconfigured users  |
| **CloudTrail**  | Log all AWS API calls                  |
| **GuardDuty**   | Detect suspicious IAM activity        |
| **Lambda**      | Process and forward detection alerts  |
| **SQS**  | Buffer between services               |
| **AWS CLI**    | Simulate attacker behavior             |
| **VS Code + Terminal** | CLI-based interaction & scripting |

---

## ⚙️ Lab Setup Overview

1. Create a low-privileged IAM user (`test-user`)  
2. Attach a misconfigured custom policy (e.g., allows `iam:PassRole`)  
3. Use `aws configure` to login as `test-user` from CLI  
4. Attempt to escalate privileges using allowed APIs  
5. Monitor the attack via CloudTrail & GuardDuty  
6. Configure Lambda to handle detection alerts  
7. Analyze the event trail and alerts in AWS  

---

## 🚨 Privilege Escalation Scenario (Example)

- IAM user allowed to perform `AttachUserPolicy`  
- Attacker uses this to attach `AdministratorAccess` policy to self  
- Result: attacker gains full root-like control  

---

## 🔍 Detection Strategy

| Tool        | What It Detects                         |
|-------------|---------------------------------------|
| **CloudTrail** | Logs API calls like `AttachUserPolicy`    |
| **GuardDuty**  | Triggers `PrivilegeEscalation:User` findings |
| **Lambda**     | Processes alerts and can trigger responses  |

---

## 🧠 Lessons Learned

- Importance of applying least privilege in IAM policies  
- How a simple policy misconfiguration can lead to full compromise  
- How GuardDuty effectively detects privilege escalation attempts  
- Leveraging Lambda for automated alert handling  
---

## 💭 Future Improvements

- Add automated remediation using EventBridge + Lambda  
- Expand detection to include role assumption abuse  
- Integrate findings with AWS Security Hub  

---
[📄 Full Documentation (CHECK ME)](https://github.com/jmcoded0/AWS-IAM-Privilege-Escalation-Detection/blob/main/documenting.md)
## 🙋🏽‍♂️ Author

**Johnson Mathew**  
Cloud Security Enthusiast | Hands-on Cybersecurity Projects  
[GitHub](https://github.com/jmcoded0) • [Twitter/X](https://twitter.com/jmcoded0)

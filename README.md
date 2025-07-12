# 🔐 AWS IAM Privilege Escalation & Detection Lab

This project simulates a real-world AWS IAM privilege escalation scenario. I created a deliberately misconfigured IAM environment, escalated privileges from a low-level user, and detected the attack using AWS native tools like **CloudTrail**, **GuardDuty**, and **Lambda**. The goal is to understand IAM policy weaknesses, simulate attacker behavior, and build a basic detection pipeline.

---

## 🧠 Objective

- Simulate IAM misconfiguration that allows **privilege escalation**
- Use AWS CLI to perform the attack (simulating an adversary)
- Detect malicious behavior with **CloudTrail** and **GuardDuty**
- Forward detection logs to **Splunk** using **Lambda + HEC**
- Document the attack path, detection events, and response

---

## 🛠️ Tools & Services Used

| Service | Purpose |
|--------|---------|
| **IAM** | Create limited & misconfigured users |
| **CloudTrail** | Log all AWS API calls |
| **GuardDuty** | Detect suspicious IAM activity |
| **Lambda** | Send alerts to Splunk |
| **SQS** (optional) | Act as a buffer between services |
| **Splunk** | Visualize and analyze detection events |
| **AWS CLI** | Simulate attacker behavior |
| **VS Code + Terminal** | For CLI-based interactions and scripting |

---

## ⚙️ Lab Setup Overview

1. Create a low-privileged IAM user (`test-user`)
2. Attach a misconfigured custom policy (e.g., allows `iam:PassRole`)
3. Use `aws configure` to login as `test-user` from CLI
4. Attempt to escalate to Admin using allowed APIs
5. Monitor the attack using CloudTrail & GuardDuty
6. Configure Lambda to forward alerts to Splunk via HEC
7. Analyze the event trail and alerts in Splunk

---

## 🚨 Privilege Escalation Scenario (Example)

- IAM user can `AttachUserPolicy`
- Attacker uses this to attach `AdministratorAccess` to self
- Now the attacker has full root-like control

---

## 🔍 Detection Strategy

| Tool | What It Detects |
|------|-----------------|
| **CloudTrail** | Logs each API call like `AttachUserPolicy` |
| **GuardDuty** | Triggers `PrivilegeEscalation:User` finding |
| **Lambda** | Parses log + sends to Splunk |
| **Splunk** | Visualizes escalation path and detection timeline |

---

## 🖼️ Screenshots

> 📸 Add screenshots of:
> - IAM policy config
> - CLI commands used
> - CloudTrail event logs
> - GuardDuty alerts
> - Splunk dashboard view

---

## 🧠 Lessons Learned

- Importance of least privilege in IAM
- How simple policy misconfig can lead to full compromise
- How GuardDuty catches privilege escalation
- How to forward AWS detections to Splunk in real time

---

## ✅ Status

- [x] Lab environment set up
- [x] Misconfigured IAM user created
- [x] Attack simulation complete
- [x] Detection + Splunk integration
- [ ] Final documentation + screenshots

---

## 💭 Future Improvements

- Add auto-remediation using EventBridge + Lambda
- Expand to detect role assumption abuse
- Integrate with AWS Security Hub

---

## 🙋🏽‍♂️ Author

**Johnson Mathew**  
Cloud Security Enthusiast | Hands-on Cybersecurity Projects  
[GitHub](https://github.com/jmcoded0) • [Twitter/X](https://twitter.com/jmcoded0)

---


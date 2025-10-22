# 🛡️ AWS IAM – Identity and Access Management

## 📍 Overview
- **IAM (Identity and Access Management)** is a **global AWS service** that helps you securely manage access to AWS services and resources.
- It controls **who is authenticated (identity)** and **what they’re authorized to do (access)**.

---

## 👤 IAM Identities

### 🔑 1. Root Account
- Created when you first set up an AWS account.
- Has **full access** to all AWS services and resources.
- **Best Practice:** Avoid using the root account for daily tasks. Enable **MFA** and store credentials securely.

### 👤 2. IAM Users
- Represent **individual people** or applications.
- Each user has:
  - A **username**
  - **Credentials** (password for console, access keys for CLI/SDK)

### 👥 3. IAM Groups
- A collection of users.
- Can be assigned to users, not other groups.
- Policies attached to a group apply to **all members**.

### 🪪 4. IAM Roles
- Identities **not tied to a specific user**.
- Used by **AWS services** to assign permissions to them.
- Example: Allow an EC2 instance to access an S3 bucket.

---

## 📜 Policies – Permissions Definition

- Policies are **JSON documents** that define what actions are allowed or denied.
- They can be attached to:
  - Users
  - Groups
  - Roles

### 🧠 Principle of Least Privilege
> Always grant only the **minimum permissions** required for a user or service to perform its tasks.

---

## 📄 Policy Structure Example

```json
{
  "Version": "2012-10-17",
  "Id": "S3-Account-Permissions",
  "Statement": [
    {
      "Sid": "1",
      "Effect": "Allow",
      "Principal": {
        "AWS": ["arn:aws:iam::1232442:root"]
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": ["arn:aws:s3:::mybucket/*"]
    }
  ]
}
```

### 🧱 Policy Elements Explained
- **Version:** Policy language version (always use `"2012-10-17"`).
- **Id:** Optional identifier for the policy.
- **Statement:** One or more individual permission blocks.
  - **Sid:** (Optional) Statement ID for reference.
  - **Effect:** `"Allow"` or `"Deny"`.
  - **Principal:** Who this applies to (user, account, role).
  - **Action:** AWS actions allowed or denied (e.g., `s3:GetObject`).
  - **Resource:** Specific resources the actions apply to.
  - **Condition:** *(Optional)* Extra conditions (e.g., IP address, MFA).

---

## 🧰 Accessing AWS

You can interact with AWS using:

1. **Management Console** – Web-based UI.
2. **CLI (Command Line Interface)** – Requires **Access Key ID** & **Secret Access Key**.
3. **SDK** – Used in code to access AWS services, also with access keys.

### 📦 CLI Essentials

- Check version:
  ```bash
  aws --version
  ```
  *(Version 2 does not require Python)*

- Configure credentials:
  ```bash
  aws configure
  ```

- Example: List IAM users:
  ```bash
  aws iam list-users
  ```

Example output:
```json
{
  "Users": [
    {
      "Path": "/",
      "UserName": "stephane",
      "UserId": "AIDA43243JKKJGDF",
      "Arn": "arn:aws:iam::432432:user/stephane",
      "CreateDate": "2023-05-22T16:28:16+00:00",
      "PasswordLastUsed": "2023-05-22T16:28:16+00:00"
    }
  ]
}
```

---

## ☁️ AWS CloudShell

- Browser-based shell preconfigured with the AWS CLI.
- Available in **specific regions only**.
- Default region is the one selected in the AWS console.
- Override region with:
  ```bash
  aws s3 ls --region eu-west-1
  ```

---

## 🛡️ IAM Security Tools
Protect account with MFA (Multi Factor Authentication) + strong Password Policy.

### 📊 Credentials Report
- Security tool that downloads a **CSV** with detailed info about IAM users:
  - Password last used
  - Access key last used
  - MFA enabled, etc.

### 🕵️ Access Advisor (Last Accessed)
- Shows **when and which services** a user last accessed and from which policy/permission.
- Helps identify **unused permissions** to tighten security.

---

### ☯️ Policy vs Permission
- A policy defines what’s possible.
- A permission is what’s granted after AWS processes all policies.

For example:
- If a user has the above S3 policy attached, they now have the permission to list objects in mybucket.
- If no policy grants that action, they do not have permission — even if the action exists in AWS.

---

## ✅ Best Practices
- Enable **MFA** (Multi-Factor Authentication) on all accounts, especially the root user.
- Set a **strong password policy** for all users.
- Apply the **least privilege principle**.
- Rotate **access keys** regularly.
- Use **roles** instead of embedding credentials in applications.

---

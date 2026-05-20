# Accessing AWS Services Using the AWS CLI

> A complete guide to installing, configuring, and using the AWS Command Line Interface (CLI) to manage AWS resources directly from your terminal, with practical examples and troubleshooting tips.

## 📋 Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [What is AWS CLI?](#what-is-aws-cli)
- [Step-by-Step Setup Guide](#step-by-step-setup-guide)
  - [1. Create an EC2 Instance](#1-create-an-ec2-instance)
  - [2. Configure Security Groups](#2-configure-security-groups)
  - [3. Connect to Your Instance](#3-connect-to-your-instance)
  - [4. Install AWS CLI](#4-install-aws-cli)
  - [5. Create IAM User and Access Keys](#5-create-iam-user-and-access-keys)
  - [6. Configure AWS CLI Credentials](#6-configure-aws-cli-credentials)
- [Common AWS CLI Commands](#common-aws-cli-commands)
- [Troubleshooting](#troubleshooting)
- [Security Best Practices](#security-best-practices)

---

## Overview

The AWS Command Line Interface (CLI) provides a unified tool for managing AWS services from your terminal. Instead of clicking through the AWS Console, you can automate operations, manage resources efficiently, and integrate AWS operations into scripts and workflows.

**What You'll Learn:**
- Launching and configuring an EC2 instance
- Installing AWS CLI on Linux systems
- Creating IAM users with appropriate permissions
- Configuring CLI credentials securely
- Using AWS CLI commands to manage resources
- Troubleshooting common connection issues

**Estimated Time:** 30-45 minutes  
**Prerequisites:** AWS account, basic Linux knowledge, SSH client (PuTTY or terminal)

---

## Prerequisites

Before you begin, ensure you have:

- ✅ An active AWS account with EC2 and IAM permissions
- ✅ Understanding of Linux terminal commands
- ✅ SSH client installed (Windows: PuTTY, Mac/Linux: built-in Terminal)
- ✅ SSH key pair for EC2 connection
- ✅ Basic knowledge of networking and security groups
- ✅ Familiarity with AWS Console navigation

**Create SSH Key Pair (if needed):**
1. Go to AWS Console → EC2 → Key Pairs
2. Click "Create key pair"
3. Name it (e.g., `my-aws-key`)
4. Download the `.pem` file safely

---

## What is AWS CLI?

### Benefits of Using AWS CLI

| Benefit | Description |
|---------|-------------|
| **Automation** | Script operations instead of manual Console clicks |
| **Speed** | Faster than web-based navigation |
| **Integration** | Embed AWS operations into applications |
| **Consistency** | Repeatable, version-controlled operations |
| **Remote Access** | Manage AWS from anywhere |
| **Scripting** | Combine multiple commands in bash scripts |

### When to Use AWS CLI

✅ **Use AWS CLI for:**
- Batch operations on multiple resources
- Automated deployments and infrastructure management
- Infrastructure-as-Code (IaC) workflows
- Scheduled maintenance tasks
- Integration with CI/CD pipelines

❌ **Use AWS Console for:**
- One-time tasks
- Initial exploration of AWS services
- Troubleshooting visual configurations
- Learning AWS features

---

## Step-by-Step Setup Guide

### 1. Create an EC2 Instance

#### What is an EC2 Instance?

EC2 (Elastic Compute Cloud) instances are virtual servers that run on AWS infrastructure. You'll use one as your "machine" to run AWS CLI commands.

#### Instructions:

1. **Sign in to AWS Console**
   - Navigate to [AWS Management Console](https://console.aws.amazon.com/)
   - Sign in with your AWS credentials

2. **Access EC2 Service**
   - Click **Services** → Search for **EC2**
   - Click on **EC2** from the results

3. **Launch Instance**
   - Click **Launch instances** button (orange button in the Instances section)
   - The Instance Launch Wizard will open

4. **Configure Instance Details**
   - **AMI Selection:** Choose **Amazon Linux 2 AMI** or **Ubuntu Server 20.04**
     - These have pre-installed tools for our use case
   - **Instance Type:** Select **t2.micro** (eligible for free tier)
   - **Region:** Choose your region (e.g., ap-south-1)

5. **Configure Security Settings**
   - **Key Pair:** Select your SSH key pair (created earlier)
   - Keep default network and subnet settings
   - Click **Next** until you reach the Security Group configuration

6. **Launch the Instance**
   - Review your settings
   - Click **Launch instances**
   - AWS will launch your instance (takes 1-2 minutes)

**Instance Status Check:**
```
Instances list → Your Instance Name
Status: running
Status Checks: 2/2 checks passed
```

---

### 2. Configure Security Groups

#### What are Security Groups?

Security groups act as firewalls that control incoming and outgoing traffic to your EC2 instance. You'll need to allow SSH (Secure Shell) traffic to connect remotely.

#### Instructions:

1. **Locate Security Group**
   - From EC2 Instances list, click on your instance
   - Scroll down to **Security** section
   - Click on the **Security groups** link

2. **Edit Inbound Rules**
   - Click **Edit inbound rules** button
   - Click **Add rule** button

3. **Configure SSH Rule**
   - **Type:** Select **SSH** (port 22)
   - **Protocol:** TCP (automatically selected)
   - **Port:** 22 (automatically filled)
   - **Source:** 
     - **Best Practice:** Enter your IP address (e.g., `192.168.1.100/32`)
     - **Quick Setup:** Select `Anywhere` (0.0.0.0/0) - less secure but easier
   - Click **Save rules**

**Visual Reference:**
```
Inbound Rules:
├── Type: SSH (Port 22)
├── Protocol: TCP
├── Port Range: 22
└── Source: Your IP (or 0.0.0.0/0)
```

**Security Note:** Restricting SSH to your IP is more secure than allowing anywhere.

---

### 3. Connect to Your Instance

#### Method 1: Using PuTTY (Windows)

1. **Convert Key File**
   - Download PuTTYgen if you don't have it
   - Open PuTTYgen
   - Load your `.pem` key file
   - Save private key as `.ppk` file

2. **Configure PuTTY Connection**
   - Open PuTTY
   - **Host Name:** Enter your instance's **Public IPv4 DNS**
     - Found in EC2 Instances list
     - Example: `ec2-43-205-138-223.ap-south-1.compute.amazonaws.com`
   - **Connection Type:** SSH
   - **Port:** 22

3. **Add Private Key**
   - Expand **Connection** → **SSH** → **Auth**
   - **Private key file for authentication:** Browse and select your `.ppk` file

4. **Connect**
   - Click **Open** to establish connection
   - Accept security warning (fingerprint check)
   - **Login as:** Enter `ec2-user` (for Amazon Linux) or `ubuntu` (for Ubuntu)

#### Method 2: Using Terminal (Mac/Linux)

1. **Set Key File Permissions**
   ```bash
   chmod 400 /path/to/my-aws-key.pem
   ```

2. **Connect via SSH**
   ```bash
   ssh -i /path/to/my-aws-key.pem ec2-user@your-instance-public-dns
   ```
   - Replace `your-instance-public-dns` with your actual DNS
   - Example: `ssh -i my-aws-key.pem ec2-user@ec2-43-205-138-223.ap-south-1.compute.amazonaws.com`

3. **Verify Connection**
   - You should see a Linux terminal prompt:
   ```bash
   [ec2-user@ip-172-31-33-196 ~]$
   ```

---

### 4. Install AWS CLI

#### What is AWS CLI?

AWS CLI is a Python-based tool that allows you to interact with AWS services using terminal commands instead of the web console.

#### Installation Instructions:

1. **Download AWS CLI v2**
   ```bash
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   ```
   - This command downloads AWS CLI installation package
   - `-o` specifies output file name

2. **Unzip the Downloaded File**
   ```bash
   unzip awscliv2.zip
   ```
   - This extracts the installation files from the zip archive

3. **Run Installation Script**
   ```bash
   sudo ./aws/install
   ```
   - `sudo` runs the command with elevated privileges
   - The installation script sets up AWS CLI on your system

4. **Verify Installation**
   ```bash
   aws --version
   ```
   - You should see output like: `aws-cli/2.X.X Python/3.X.X Linux/5.X.X`

**Expected Output:**
```bash
[ec2-user@ip-172-31-33-196 ~]$ aws --version
aws-cli/2.X.X Python/3.X.X Linux/5.X.X
```

---

### 5. Create IAM User and Access Keys

#### What is IAM?

AWS Identity and Access Management (IAM) allows you to create users and manage permissions. Instead of using your root account, create an IAM user with specific permissions.

#### Why Not Use Root Account?

- ❌ Root account has unlimited access (security risk)
- ✅ IAM users have limited permissions (security best practice)
- ✅ You can revoke access without changing root credentials

#### Instructions:

1. **Open IAM Service**
   - AWS Console → Services → **IAM**
   - Click **Users** in the left menu

2. **Create New User**
   - Click **Create user** button
   - **User name:** Enter a name (e.g., `aws-cli-user`)
   - **Check** "Provide user access to AWS Management Console" (optional)
   - Click **Next**

3. **Set Permissions**
   - Select **Attach policies directly**
   - Search for and select **AmazonEC2FullAccess**
     - This gives the user full EC2 permissions
   - You can restrict permissions later for security
   - Click **Next**

4. **Review and Create**
   - Review user settings
   - Click **Create user**

5. **Generate Access Keys**
   - Click on your newly created user name
   - Go to **Security credentials** tab
   - Under **Access keys**, click **Create access key**
   - **Use case:** Select **Command Line Interface (CLI)**
   - Check confirmation checkbox
   - Click **Create access key**

6. **Save Your Keys**
   - **Access Key ID:** Copy and save securely
   - **Secret Access Key:** Copy and save securely
   - ⚠️ **Critical:** This is your only chance to view the secret key!
   - You can download `.csv` file with both keys

**Key Information to Save:**
```
Access Key ID:     AKIA3FLDXS MMHGOE5NEO
Secret Access Key: QC9dEHQgTFt+yKRYq0q2CH8ihP0DN3YIVtVCrkQO
Region:            ap-south-1
Output format:     json
```

---

### 6. Configure AWS CLI Credentials

#### What Does Configuration Do?

Configuration tells AWS CLI which credentials to use when you run commands.

#### Instructions:

1. **Run Configuration Command**
   ```bash
   aws configure
   ```
   - This interactive command will ask for credentials

2. **Enter Your Credentials**
   ```
   AWS Access Key ID [None]: AKIA3FLDXS MMHGOE5NEO
   AWS Secret Access Key [None]: QC9dEHQgTFt+yKRYq0q2CH8ihP0DN3YIVtVCrkQO
   Default region name [None]: ap-south-1
   Default output format [None]: json
   ```

3. **Verify Configuration**
   ```bash
   aws sts get-caller-identity
   ```
   - This command returns information about your configured user
   - Successful output confirms credentials are working:
   ```json
   {
       "UserId": "AIDAI...",
       "Account": "123456789012",
       "Arn": "arn:aws:iam::123456789012:user/aws-cli-user"
   }
   ```

4. **Configuration Files Location**
   - Credentials stored in: `~/.aws/credentials`
   - Configuration stored in: `~/.aws/config`
   - **Note:** Keep these files private; don't share them!

---

## Common AWS CLI Commands

### EC2 Commands

#### List All Running Instances
```bash
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,InstanceType,State.Name]' --output table
```
- Lists instance IDs, types, and current states
- `--query` filters output to show specific fields
- `--output table` formats as readable table

#### Start an Instance
```bash
aws ec2 start-instances --instance-ids i-0123456789abcdef0 --region ap-south-1
```
- Starts a stopped EC2 instance
- Replace instance ID with your actual instance ID

#### Stop an Instance
```bash
aws ec2 stop-instances --instance-ids i-0123456789abcdef0 --region ap-south-1
```
- Stops a running instance (doesn't terminate)
- Data persists when restarted

#### Terminate an Instance
```bash
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0 --region ap-south-1
```
- ⚠️ **Caution:** This deletes the instance permanently
- Data is lost

### S3 Commands

#### List S3 Buckets
```bash
aws s3 ls
```
- Lists all your S3 buckets

#### Upload File to S3
```bash
aws s3 cp /path/to/local/file s3://my-bucket/remote-path/
```
- Uploads a file from your local system to S3

#### Download File from S3
```bash
aws s3 cp s3://my-bucket/remote-file /path/to/local/destination/
```
- Downloads a file from S3 to your local system

### IAM Commands

#### List All Users
```bash
aws iam list-users
```
- Shows all IAM users in your AWS account

#### Get Current User Information
```bash
aws iam get-user
```
- Shows details about the currently authenticated user

### General Helpful Commands

#### Get Command Help
```bash
aws ec2 describe-instances help
```
- Shows detailed documentation for any AWS CLI command

#### Test Connection
```bash
aws sts get-caller-identity
```
- Verifies credentials are configured correctly

---

## Troubleshooting

### Problem: "Unable to locate credentials"

**Cause:** AWS CLI configuration not set or credentials missing

**Solutions:**
1. Re-run `aws configure` with correct credentials
2. Verify files exist: `cat ~/.aws/credentials`
3. Check environment variables: `echo $AWS_ACCESS_KEY_ID`

### Problem: "An error occurred (UnauthorizedOperation)"

**Cause:** IAM user doesn't have required permissions

**Solutions:**
1. Attach correct IAM policy to user
2. Verify user has `AmazonEC2FullAccess` or required permissions
3. Wait 1-2 minutes for IAM policy to take effect

### Problem: "SSH Access Denied"

**Cause:** Wrong key file or security group not configured

**Solutions:**
1. Verify security group allows SSH (port 22)
2. Check key file permissions: `chmod 400 key.pem`
3. Confirm you're using correct key pair for instance
4. Try connecting again: `ssh -i key.pem ec2-user@public-dns`

### Problem: "Instance not reachable"

**Cause:** Instance not yet running or security configuration issue

**Solutions:**
1. Verify instance status: `aws ec2 describe-instances`
2. Check security group inbound rules allow SSH
3. Confirm your IP has access (firewall/VPN issues)
4. Wait for instance to fully boot (status checks should pass)

---

## Security Best Practices

### Credential Management
✅ **Do:**
- Store credentials in `~/.aws/credentials` (AWS CLI handles automatically)
- Use separate IAM users for different applications
- Rotate access keys regularly (annually minimum)
- Limit permissions using least-privilege principle
- Use temporary credentials (STS) for short-lived sessions

❌ **Don't:**
- Hardcode credentials in scripts or files
- Share access keys or secret keys
- Commit credentials to version control
- Use root account credentials for CLI access
- Leave credentials in command history

### Instance Security
✅ **Do:**
- Restrict SSH to your IP address (0.0.0.0/0 is dangerous)
- Use strong, unique SSH key pairs
- Keep operating system updated
- Disable password-based SSH authentication
- Use security groups effectively

❌ **Don't:**
- Allow SSH from anywhere (0.0.0.0/0)
- Reuse SSH keys across instances
- Leave instances running 24/7 if not needed
- Ignore security group configuration
- Use weak or default security settings

### Regional Awareness
- Always specify region in commands: `--region ap-south-1`
- Different regions have different resource limits
- Data transfer between regions incurs charges
- Choose region closest to your users for performance

---

## Advanced Topics

### Create Bash Scripts with AWS CLI
```bash
#!/bin/bash
# List running instances and display their details

RUNNING_INSTANCES=$(aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

echo "Running Instances: $RUNNING_INSTANCES"

for instance in $RUNNING_INSTANCES; do
  aws ec2 describe-instances --instance-ids $instance
done
```

### Use AWS CLI with CloudWatch
```bash
# Monitor EC2 CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2024-04-17T00:00:00Z \
  --end-time 2024-04-18T00:00:00Z \
  --period 3600 \
  --statistics Average
```

### Automate Resource Management
```bash
# Stop all instances with specific tag
aws ec2 describe-instances \
  --filters "Name=tag:Environment,Values=Development" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text | \
  xargs -I {} aws ec2 stop-instances --instance-ids {}
```

---

## Conclusion

You now have AWS CLI set up and ready to automate AWS resource management! This powerful tool enables:
- Automated infrastructure management
- Integration with deployment pipelines
- Faster troubleshooting and monitoring
- Reproducible infrastructure-as-code workflows

For more advanced usage, explore the [AWS CLI Documentation](https://docs.aws.amazon.com/cli/).

---

## Additional Resources

- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [AWS IAM Documentation](https://docs.aws.amazon.com/iam/)
- [AWS Security Best Practices](https://aws.amazon.com/architecture/security-identity-compliance/)

---

**Author:** Pavithra Srinivasan  
**Last Updated:** April 2024  
**AWS Services:** EC2, IAM, CLI  
**Region:** Asia Pacific (Mumbai) - ap-south-1


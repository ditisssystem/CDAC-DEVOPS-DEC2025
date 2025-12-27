# SESSION 11 – LAB MANUAL (PART 1)

## LAB 11.1 – Cloud API Integration on AWS using AWS Console + AWS CLI

_(EXTREME DETAIL | Self-Study | No Prior Labs Assumed)_

## 1. Lab Objective

After completing this lab, the student will be able to:

- Understand what **Cloud API Integration** means in AWS
- Install and configure **AWS CLI** on their local machine
- Launch an **EC2 Linux instance** using:

  - AWS Management Console
  - AWS CLI (API-based approach)

- Verify that both methods create identical infrastructure

## 2. What You Will Build

You will create:

- One **Linux EC2 instance** using AWS Console
- One **Linux EC2 instance** using AWS CLI (API)
- Ability to connect to EC2 using **SSH**

## 3. Prerequisites (VERY IMPORTANT – READ CAREFULLY)

### 3.1 AWS Account

- A valid AWS account
- Billing enabled
- AWS Free Tier available

### 3.2 Local Machine Requirements

| Item       | Requirement                        |
| ---------- | ---------------------------------- |
| OS         | Windows / macOS / Linux            |
| Internet   | Mandatory                          |
| Terminal   | Command Prompt / Terminal          |
| SSH Client | Installed (default on Linux/macOS) |

## 4. AWS Concepts You Must Know (Quick Explanation)

| Term           | Meaning                         |
| -------------- | ------------------------------- |
| EC2            | Virtual machine in AWS          |
| AMI            | OS image for EC2                |
| Key Pair       | Used for secure login           |
| Security Group | Firewall rules                  |
| AWS CLI        | Command-line access to AWS APIs |

## 5. Step-by-Step Instructions (AWS CONSOLE METHOD)

### STEP 1: Login to AWS Console

1. Open browser
2. Go to: [https://aws.amazon.com](https://aws.amazon.com)
3. Click **Sign in to the Console**
4. Enter AWS credentials

### STEP 2: Select AWS Region

1. Top-right corner of console
2. Select: **Asia Pacific (Mumbai) – ap-south-1**

You MUST stay in the same region for the entire lab.

### STEP 3: Open EC2 Dashboard

1. In the AWS search bar, type **EC2**
2. Click **EC2**
3. EC2 Dashboard opens

### STEP 4: Start Launching EC2 Instance

1. Click **Launch instance**
2. Click **Launch instance** again

### STEP 5: Choose AMI (Operating System)

1. Under **Application and OS Images**
2. Select:

   - **Amazon Linux**
   - Version: **Amazon Linux 2023**

3. Confirm it is **Free Tier eligible**

AMI decides the operating system of your VM.

### STEP 6: Choose Instance Type

1. Select **t2.micro**
2. Ensure **Free Tier eligible** is shown

### STEP 7: Create Key Pair

1. Click **Create new key pair**
2. Enter:

   - Name: `session11-key`
   - Key type: RSA
   - File format: `.pem`

3. Click **Create key pair**
4. File downloads automatically

Store this file safely. AWS will not give it again.

### STEP 8: Configure Network

1. VPC: Default
2. Subnet: Default
3. Auto-assign public IP: **Enable**

### STEP 9: Configure Security Group (Firewall)

1. Select **Create security group**
2. Name: `session11-sg`
3. Description: Allow SSH
4. In inbound rules:

   - Type: SSH
   - Port: 22
   - Source: My IP

This allows SSH access from your system.

### STEP 10: Configure Storage

1. Root volume size: **8 GB**
2. Type: gp3
3. Keep defaults

### STEP 11: Launch Instance

1. Review all settings
2. Click **Launch instance**

### STEP 12: Verify Instance

1. Go to **EC2 → Instances**
2. Check:

   - Instance state: **Running**
   - Status checks: **2/2 passed**

## 6. Connect to EC2 (Console-Launched Instance)

### STEP 13: Get Public IP

1. Select EC2 instance
2. Copy **Public IPv4 address**

### STEP 14: Connect Using SSH

```bash
chmod 400 session11-key.pem
ssh -i session11-key.pem ec2-user@<PUBLIC_IP>
```

If you see Amazon Linux prompt, connection is successful.

## 7. Step-by-Step Instructions (AWS CLI / API METHOD)

This is the **ACTUAL Cloud API Integration** part.

### STEP 15: Install AWS CLI

#### Linux / macOS

```bash
aws --version
```

If not installed:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip awscliv2.zip
sudo ./aws/install
```

### STEP 16: Configure AWS CLI

```bash
aws configure
```

Enter:

- Access Key ID: `<from AWS IAM>`
- Secret Access Key: `<from AWS IAM>`
- Region: `ap-south-1`
- Output format: `json`

### STEP 17: Verify CLI Access

```bash
aws ec2 describe-instances
```

If output appears, CLI is working.

### STEP 18: Launch EC2 via AWS API (CLI)

```bash
aws ec2 run-instances \
--image-id ami-0f5ee92e2d63afc18 \
--instance-type t2.micro \
--key-name session11-key \
--security-group-ids <SECURITY_GROUP_ID> \
--count 1
```

This command directly calls AWS EC2 APIs.

### STEP 19: Verify CLI-Created Instance

```bash
aws ec2 describe-instances
```

Check:

- Instance state = running
- Instance ID exists

## 8. Verification / Acceptance Criteria

- EC2 created via Console
- EC2 created via CLI
- SSH works for both
- No manual UI used for CLI instance

## 9. Common Mistakes & Fixes

| Problem           | Solution              |
| ----------------- | --------------------- |
| Permission denied | Check key permissions |
| SSH timeout       | Check security group  |
| Wrong region      | Match console & CLI   |

## 10. Cleanup (MANDATORY – COST CONTROL)

1. Go to **EC2 → Instances**
2. Select both instances
3. Click **Instance state → Terminate**
4. Confirm termination

## 11. Learning Outcomes

After this lab, you can:

- Use AWS as an API-driven platform
- Launch infrastructure programmatically
- Understand cloud automation fundamentals

# SESSION 11 – LAB MANUAL (PART 2)

# LAB 11.2 – DC → DR Migration (On-Prem / Local Server to AWS EC2)

_(EXTREMELY DETAILED | SELF-STUDY | AWS-ONLY)_

## 1. Lab Objective

By the end of this lab, the student will be able to:

- Understand **DC/DR migration concepts**
- Create an **AWS EC2 Linux server** to act as a **DR server**
- Simulate an **on-premises (DC) server**
- Migrate application data from DC to AWS EC2 using **rsync**
- Validate data integrity after migration

## 2. What You Will Build

You will create:

- One **Linux EC2 instance on AWS** (DR Server)
- One **local Linux system** (DC Server – your laptop or VM)
- A **data migration pipeline** from DC → AWS

## 3. Important Concepts (Read Before Starting)

### What is DC/DR Migration?

- **DC (Data Center)** → Primary production system
- **DR (Disaster Recovery)** → Backup system in another location
- Migration ensures **business continuity**

### Migration Type Used in This Lab

- **Cold Migration**

  - Application stopped
  - Data copied
  - Application started on DR

## 4. Prerequisites (VERY IMPORTANT)

### 4.1 AWS Requirements

- AWS account
- Region: **ap-south-1 (Mumbai)**

### 4.2 Local Machine Requirements

- Linux system OR
- Windows/macOS with:

  - WSL / Linux VM / Git Bash

- SSH client available

### 4.3 Knowledge (Basic)

- Linux commands: `ls`, `mkdir`, `cat`
- SSH basics

## 5. Architecture Overview (Textual)

```
[ Local DC Server ]
        |
        |  (rsync over SSH)
        |
[ AWS EC2 – DR Server ]
```

## 6. Step-by-Step Instructions

### STEP 1: Launch DR EC2 Instance on AWS

> Even if you did this earlier, FOLLOW THESE STEPS AGAIN.

#### 1. Login to AWS Console

- [https://aws.amazon.com](https://aws.amazon.com) → Sign in

#### 2. Open EC2

- Search → EC2 → Open EC2 Dashboard

#### 3. Launch Instance

- Click **Launch Instance**

#### 4. Choose AMI

- Amazon Linux 2023
- Free Tier eligible

#### 5. Choose Instance Type

- `t2.micro`

#### 6. Create Key Pair

- Name: `dr-migration-key`
- Type: RSA
- Format: `.pem`
- Download and save securely

#### 7. Security Group

- Name: `dr-migration-sg`
- Inbound rules:

  - SSH (22) → My IP

#### 8. Storage

- 8 GB (default)

#### 9. Launch Instance

### STEP 2: Connect to DR EC2

```bash
chmod 400 dr-migration-key.pem
ssh -i dr-migration-key.pem ec2-user@<EC2_PUBLIC_IP>
```

You should see:

```
[ec2-user@ip-xxx ~]$
```

### STEP 3: Prepare DR Server Directory

On the EC2 instance:

```bash
sudo mkdir /dr-data
sudo chown ec2-user:ec2-user /dr-data
```

### STEP 4: Prepare DC (Local) Server Data

On your **local machine**:

```bash
mkdir ~/dc-data
cd ~/dc-data
```

Create sample files:

```bash
echo "Customer records" > customers.txt
echo "Order history" > orders.txt
echo "Application config" > app.conf
```

Verify:

```bash
ls
```

### STEP 5: Install rsync (If Not Installed)

```bash
sudo apt install rsync -y
```

(macOS users: rsync already exists)

### STEP 6: Perform DC → DR Migration

Run from **local DC system**:

```bash
rsync -av ~/dc-data ec2-user@<EC2_PUBLIC_IP>:/dr-data
```

Explanation:

- `-a` → archive (preserves permissions)
- `-v` → verbose output
- SSH is used automatically

### STEP 7: Verify Migration on DR Server

SSH into EC2:

```bash
ssh -i dr-migration-key.pem ec2-user@<EC2_PUBLIC_IP>
```

Check files:

```bash
ls /dr-data/dc-data
cat /dr-data/dc-data/customers.txt
```

## 7. Verification / Acceptance Criteria

- EC2 instance running
- SSH connection successful
- Files visible on DR server
- File contents intact

## 8. Common Mistakes & Troubleshooting

| Issue             | Cause           | Fix                  |
| ----------------- | --------------- | -------------------- |
| Permission denied | Key permissions | `chmod 400`          |
| Timeout           | Security Group  | Allow SSH            |
| Files missing     | Wrong path      | Recheck rsync source |

## 9. Cleanup (IMPORTANT)

1. Terminate EC2 instance
2. Delete key pair (optional)
3. Remove local test data

## 10. Learning Outcomes

After this lab, you understand:

- DC vs DR concepts
- Real migration workflow
- Secure data transfer to AWS

# LAB 11.3 – DC/DR Storage Synchronization on AWS (Continuous Sync)

_(EXTREMELY DETAILED | SELF-STUDY)_

## 1. Lab Objective

By the end of this lab, the student will be able to:

- Implement **continuous data synchronization**
- Use **cron + rsync**
- Ensure DR data is always updated automatically

## 2. What You Will Build

- Continuous sync from DC → AWS EC2
- No manual intervention required

## 3. Prerequisites

- Completion of **Lab 11.2**
- EC2 instance running
- SSH access working

## 4. Why Storage Synchronization Is Needed

- Migration is **one-time**
- Sync ensures **ongoing protection**
- Used in **real DR systems**

## 5. Step-by-Step Instructions

### STEP 1: Modify DC Data

On local machine:

```bash
echo "New order added" >> ~/dc-data/orders.txt
```

### STEP 2: Create Cron Job on DC Server

Edit crontab:

```bash
crontab -e
```

Add:

```bash
*/5 * * * * rsync -av ~/dc-data ec2-user@<EC2_PUBLIC_IP>:/dr-data
```

Explanation:

- Every 5 minutes
- Automatically syncs changes

Save and exit.

### STEP 3: Wait for Sync Cycle

Wait **5 minutes**.

### STEP 4: Verify on DR Server

SSH into EC2:

```bash
cat /dr-data/dc-data/orders.txt
```

You should see updated content.

## 6. Verification / Acceptance Criteria

- Cron job exists
- Data syncs automatically
- No manual command required

## 7. Failure Simulation (IMPORTANT)

Delete file on DR:

```bash
rm /dr-data/dc-data/app.conf
```

Wait 5 minutes.

Verify:

```bash
ls /dr-data/dc-data
```

File restored automatically

## 8. Cleanup

- Remove cron job
- Terminate EC2 instance

## 9. Learning Outcomes

After this lab, you understand:

- Continuous DR sync
- Automation using cron
- Real-world DR patterns

# SESSION 12 – LAB MANUAL (PART 3)

# LAB 12.1 – Configuration Management on AWS using Puppet (From Scratch)

_(EXTREMELY DETAILED | SELF-STUDY | AWS-ONLY | NO ASSUMPTIONS)_

## 1. Lab Objective

By the end of this lab, the student will be able to:

- Understand **why configuration management is needed**
- Launch **two AWS EC2 Linux instances**

  - One as **Puppet Server**
  - One as **Puppet Agent (Client)**

- Install and configure **Puppet**
- Automatically install and manage software on EC2
- Verify configuration enforcement

## 2. What You Will Build

You will create:

- 1 EC2 instance → **Puppet Server**
- 1 EC2 instance → **Puppet Agent**
- Automated configuration where:

  - Apache is installed
  - Apache service is running
  - Configuration is enforced automatically

## 3. Why Configuration Management Is Important (Concept)

Without configuration management:

- Manual installation
- Human errors
- Inconsistent environments

With Puppet:

- **Infrastructure as Code**
- Repeatable builds
- Automated enforcement

## 4. Prerequisites (READ CAREFULLY)

### 4.1 AWS Requirements

- AWS account
- Region: **ap-south-1**
- Free Tier available

### 4.2 Local Machine Requirements

- Terminal / Command Prompt
- SSH client
- Internet connectivity

### 4.3 Knowledge (Minimum)

- Basic Linux commands
- SSH access

## 5. Architecture Overview (Textual)

```
[ Puppet Server (EC2) ]
        |
        |  (Puppet Agent Communication)
        |
[ Puppet Agent (EC2) ]
```

## 6. Step-by-Step Instructions

## PART A – Launch Puppet Server EC2

### STEP 1: Login to AWS Console

- [https://aws.amazon.com](https://aws.amazon.com)
- Sign in to Console

### STEP 2: Open EC2 Dashboard

- Search → EC2
- Click **EC2**

### STEP 3: Launch EC2 Instance (Puppet Server)

1. Click **Launch instance**
2. Name: `puppet-server`
3. AMI:

   - Amazon Linux 2023

4. Instance Type:

   - `t2.micro`

5. Key Pair:

   - Create new
   - Name: `puppet-key`
   - Download `.pem`

6. Security Group:

   - Name: `puppet-sg`
   - Inbound Rules:

     - SSH (22) → My IP
     - Custom TCP → Port **8140** → Anywhere (Puppet)

7. Storage:

   - 8 GB

8. Click **Launch instance**

### STEP 4: Verify Puppet Server Instance

- Instance state: **Running**
- Status checks: **2/2**

### STEP 5: Connect to Puppet Server

```bash
chmod 400 puppet-key.pem
ssh -i puppet-key.pem ec2-user@<PUPPET_SERVER_PUBLIC_IP>
```

## PART B – Install Puppet Server

### STEP 6: Update System

```bash
sudo dnf update -y
```

### STEP 7: Install Puppet Server

```bash
sudo dnf install -y https://yum.puppet.com/puppet7-release-el-9.noarch.rpm
sudo dnf install -y puppetserver
```

### STEP 08: Start and Enable Puppet Server

```bash
sudo systemctl start puppetserver
sudo systemctl enable puppetserver
```

Verify:

```bash
sudo systemctl status puppetserver
```

## PART C – Launch Puppet Agent EC2

### STEP 9: Launch EC2 Instance (Agent)

Repeat **EC2 launch steps**, with changes:

- Name: `puppet-agent`
- Same AMI and instance type
- Same key pair: `puppet-key`
- Same security group: `puppet-sg`

### STEP 10: Connect to Puppet Agent

```bash
ssh -i puppet-key.pem ec2-user@<PUPPET_AGENT_PUBLIC_IP>
```

## PART D – Install Puppet Agent

### STEP 11: Install Puppet Agent

```bash
sudo dnf install -y https://yum.puppet.com/puppet7-release-el-9.noarch.rpm
sudo dnf install -y puppet-agent
```

### STEP 12: Configure Puppet Agent

Edit config file:

```bash
sudo vi /etc/puppetlabs/puppet/puppet.conf
```

Add under `[main]`:

```
server = <PUPPET_SERVER_PRIVATE_IP>
```

Save and exit.

## PART E – Certificate Signing (VERY IMPORTANT)

### STEP 13: Start Puppet Agent

```bash
sudo /opt/puppetlabs/bin/puppet agent --test
```

You will see:

```
Certificate request sent
```

### STEP 14: Sign Certificate on Puppet Server

On **Puppet Server EC2**:

```bash
sudo /opt/puppetlabs/bin/puppetserver ca list
sudo /opt/puppetlabs/bin/puppetserver ca sign --all
```

### STEP 15: Re-run Agent

On **Puppet Agent**:

```bash
sudo /opt/puppetlabs/bin/puppet agent --test
```

Certificate signed successfully

## PART F – Enforce Configuration (Apache Installation)

### STEP 16: Create Puppet Manifest

On **Puppet Server**:

```bash
sudo vi /etc/puppetlabs/code/environments/production/manifests/site.pp
```

Add:

```puppet
node default {
  package { 'httpd':
    ensure => installed,
  }

  service { 'httpd':
    ensure => running,
    enable => true,
  }
}
```

Save and exit.

### STEP 17: Apply Configuration on Agent

```bash
sudo /opt/puppetlabs/bin/puppet agent --test
```

### STEP 18: Verify Apache on Agent

```bash
systemctl status httpd
```

Open browser:

```
http://<PUPPET_AGENT_PUBLIC_IP>
```

Apache test page visible

## 7. Verification / Acceptance Criteria

- Puppet Server running
- Puppet Agent connected
- Apache installed automatically
- Service running without manual install

## 8. Common Errors & Fixes

| Issue                | Cause      | Fix            |
| -------------------- | ---------- | -------------- |
| Agent not connecting | Wrong IP   | Use private IP |
| Port blocked         | SG missing | Open 8140      |
| Cert pending         | Not signed | Sign CA        |

## 9. Cleanup (MANDATORY)

1. Terminate both EC2 instances
2. Delete security group (optional)
3. Delete key pair (optional)

## 10. Learning Outcomes

After this lab, the student understands:

- Configuration management concepts
- Puppet architecture
- Automated server provisioning
- Real-world DevOps workflows

# SESSION 13 – LAB MANUAL (PART 4)

## LAB 13.1 – Centralized Logging on AWS EC2 (Linux → Central Log Server)

_(EXTREMELY DETAILED | SELF-STUDY | AWS-ONLY)_

### 1. Objective

- Centralize logs from multiple Linux EC2 instances to a single **Log Server**.
- Understand why centralized logging is critical for operations and audits.

### 2. What You Will Build

- **Log Server EC2** (collector)
- **App Server EC2** (sender)
- Automated log shipping using `rsync + cron`

### 3. Prerequisites

- AWS account (Free Tier)
- Region: **ap-south-1**
- Local SSH client

### 4. Architecture (Textual)

```
[ App EC2 ] --rsync--> [ Log EC2 ]
```

### 5. Step-by-Step

#### STEP A: Launch Log Server EC2

1. AWS Console → **EC2** → **Launch instance**
2. Name: `log-server`
3. AMI: **Amazon Linux 2023**
4. Instance type: `t2.micro`
5. Key pair: create `log-key.pem`
6. Security group:

   - SSH (22) → My IP

7. Launch → wait for **2/2 checks**

#### STEP B: Launch App Server EC2

Repeat above with:

- Name: `app-server`
- Same key pair & SG

#### STEP C: Prepare Log Server

```bash
chmod 400 log-key.pem
ssh -i log-key.pem ec2-user@<LOG_SERVER_PUBLIC_IP>
sudo mkdir /central-logs
sudo chown ec2-user:ec2-user /central-logs
exit
```

#### STEP D: Send Logs from App Server

```bash
ssh -i log-key.pem ec2-user@<APP_SERVER_PUBLIC_IP>
sudo yum install -y rsync
rsync -av /var/log ec2-user@<LOG_SERVER_PRIVATE_IP>:/central-logs
```

#### STEP E: Automate with Cron (App Server)

```bash
crontab -e
```

Add:

```
*/10 * * * * rsync -av /var/log ec2-user@<LOG_SERVER_PRIVATE_IP>:/central-logs
```

### 6. Verification

- Files appear under `/central-logs`
- Updates occur every 10 minutes

### 7. Cleanup

- Terminate both EC2 instances

## LAB 13.2 – Nagios Monitoring on AWS (Linux EC2)

### 1. Objective

- Install **Nagios** and monitor a Linux EC2 host (CPU, disk, services).

### 2. Build

- **Nagios Server EC2**
- **Client EC2**

### 3. Steps

#### A. Launch Two EC2 Instances

- `nagios-server`, `nagios-client`
- Amazon Linux 2023, `t2.micro`
- SG:

  - SSH (22) → My IP
  - HTTP (80) → My IP

#### B. Install Nagios (Server)

```bash
ssh -i nagios-key.pem ec2-user@<SERVER_IP>
sudo yum install -y nagios nagios-plugins-all
sudo systemctl start nagios
sudo systemctl enable nagios
```

Access:

```
http://<SERVER_IP>/nagios
```

#### C. Configure Client Checks

- Install NRPE on client

```bash
sudo yum install -y nrpe nagios-plugins-all
sudo systemctl start nrpe
```

- Add host in server config

```bash
sudo vi /etc/nagios/conf.d/client.cfg
```

### 4. Verification

- Client visible in Nagios UI
- CPU/Disk status shown

### 5. Cleanup

- Terminate EC2 instances

## LAB 13.3 – Nagios Monitoring on AWS (Windows EC2)

### 1. Objective

- Monitor **Windows EC2** from Nagios (Linux).

### 2. Steps (VERY DETAILED)

#### A. Launch Windows EC2

1. EC2 → Launch instance
2. AMI: **Windows Server 2019 Base**
3. Instance type: `t2.micro`
4. SG:

   - RDP (3389) → My IP
   - NRPE (5666) → Nagios SG

5. Launch

#### B. Connect via RDP

- EC2 → **Connect** → RDP
- Decrypt password using `.pem`
- Login as **Administrator**

#### C. Install NSClient++

1. Download NSClient++
2. Enable NRPE
3. Open port 5666 in Windows Firewall

#### D. Add Windows Host in Nagios

- Configure host & services
- Restart Nagios

### 3. Verification

- Windows host visible
- Disk/CPU metrics shown

### 4. Cleanup

- Terminate Windows EC2

## LAB 13.4 – Prometheus Monitoring on AWS EC2

### 1. Objective

- Deploy **Prometheus** and collect system metrics.

### 2. Steps

#### A. Launch EC2

- Name: `prometheus-server`
- Amazon Linux 2023

#### B. Install Docker

```bash
sudo yum install -y docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
```

#### C. Run Prometheus

```bash
docker run -d -p 9090:9090 prom/prometheus
```

Access:

```
http://<EC2_IP>:9090
```

### 3. Verification

- Metrics visible
- PromQL queries work

### 4. Cleanup

- Stop container
- Terminate EC2

## LAB 13.5 – Bottleneck Identification on AWS EC2

### Objective

Identify CPU, memory, disk, and network bottlenecks.

### Steps

```bash
top
htop
free -m
df -h
iostat
```

Simulate load:

```bash
yes > /dev/null &
```

### Verification

- High CPU visible
- Root cause identified

## LAB 13.6 – Auto-Scaling & Auto-Healing on AWS

### Objective

Implement **self-healing infrastructure**.

### Steps

1. Create **Launch Template**
2. Create **Auto Scaling Group**
3. Attach **CloudWatch health checks**
4. Manually terminate an instance

### Verification

- New EC2 launched automatically
- Service restored without manual action

## LAB 13.7 – Zero-Downtime Deployment on AWS

### Objective

Deploy updates with **no downtime**.

### Steps

1. Create **Application Load Balancer**
2. Register two EC2 instances
3. Perform **rolling update**
4. Observe traffic continuity

### Verification

- Application always reachable
- No request failures

## SESSION 13 – FINAL LEARNING OUTCOMES

After Part 4, a student can:

- Implement centralized logging
- Monitor Linux & Windows on AWS
- Use Prometheus for modern observability
- Identify performance bottlenecks
- Implement auto-healing & scaling
- Deploy with zero downtime

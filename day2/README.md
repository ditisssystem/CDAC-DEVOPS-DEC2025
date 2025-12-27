# **Session 11 & 12 – Cloud API Integration and DC/DR Migration (AWS Focus)**

## **Learning Objectives**

By the end of Sessions 11 & 12, you will be able to:

- Understand how cloud APIs enable automation, DR, and large-scale infrastructure control
- Design and implement DC/DR strategies using AWS
- Migrate physical and virtual servers into AWS
- Synchronize storage between on-premises DC and AWS
- Bootstrap and use configuration management tools (Chef / Puppet) for cloud migration
- Build highly available and automated cloud infrastructure

## **Prerequisites**

Students should already know:

- Basic Linux administration (SSH, services, filesystems)
- Networking fundamentals (IP, DNS, ports)
- AWS basics: EC2, VPC, Security Groups
- Git basics (optional but recommended)

### **AWS Account Requirements**

- Active AWS account
- IAM user with:

  - EC2 full access
  - VPC full access
  - S3 full access
  - IAM role creation permissions

- AWS CLI installed and configured

## **Theory – Session 11 & 12**

## **1. Cloud API Integration**

### **What is Cloud API Integration?**

Cloud API integration refers to **programmatic interaction with cloud services** instead of manual console operations.

In AWS:

- Every console action is backed by an API
- AWS CLI, SDKs (Python boto3, Java SDK), Terraform all use AWS APIs

### **Why Cloud APIs are Critical for DC/DR**

- Automated disaster recovery
- Infrastructure as Code
- Faster failover
- No manual intervention during outages
- Enables auto-healing and scaling

### **Examples of AWS APIs**

| API Purpose          | AWS Service    |
| -------------------- | -------------- |
| Create servers       | EC2 API        |
| Attach storage       | EBS API        |
| Replicate data       | S3 API         |
| Configure networking | VPC API        |
| Monitor systems      | CloudWatch API |

### **Practical Example – Launch EC2 Using AWS CLI**

```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --key-name mykeypair \
  --security-group-ids sg-123456 \
  --subnet-id subnet-123456
```

This demonstrates:

- Infrastructure creation using APIs
- Repeatability
- Automation readiness

## **2. DC/DR Migration**

### **What is DC/DR Migration?**

DC/DR migration is the process of:

- Moving workloads from on-premises data center (DC)
- Creating a Disaster Recovery (DR) site in the cloud
- Ensuring business continuity

### **Common Migration Strategies**

| Strategy   | Meaning                   |
| ---------- | ------------------------- |
| Rehost     | Lift and shift            |
| Replatform | Minor cloud optimizations |
| Refactor   | Cloud-native redesign     |
| Hybrid DR  | Partial cloud DR          |

## **AWS Services Used for DC/DR**

- EC2 – Compute
- EBS – Block storage
- S3 – Object storage
- AWS DataSync – Data synchronization
- AWS Application Migration Service (MGN) – Server migration
- Route 53 – DNS failover

![Image](https://d2908q01vomqb2.cloudfront.net/fc074d501302eb2b93e2554793fcaf50b3bf7291/2021/05/13/Figure-2.-Pilot-light-DR-strategy.png)

![Image](https://d2908q01vomqb2.cloudfront.net/9e6a55b6b4563e652a23be9d623ca5055c356940/2021/07/08/migrating-cms-on-premise-workload-aws-mgn-figure-1.png)

![Image](https://d2908q01vomqb2.cloudfront.net/fc074d501302eb2b93e2554793fcaf50b3bf7291/2021/05/13/Figure-3.-Warm-standby-DR-strategy.png)

## **3. DC/DR Storage Synchronization**

### **Why Storage Sync is Critical**

- Data consistency
- Minimal RPO (Recovery Point Objective)
- Seamless failover

### **AWS Storage Synchronization Options**

| Tool           | Use Case                     |
| -------------- | ---------------------------- |
| AWS DataSync   | File and object sync         |
| S3 Replication | Bucket-to-bucket replication |
| EBS Snapshots  | Block-level backup           |
| rsync          | File-level Linux sync        |

### **Example – rsync Based Sync**

```bash
rsync -avz /data/ user@aws-ec2:/data/
```

## **Suggested Teaching / Practical Focus**

## **4. Bootstrapping Chef / Puppet Server**

### **What is Bootstrapping?**

Bootstrapping means:

- Installing configuration management agent
- Registering the node with the master server
- Enforcing desired state

## **Chef Bootstrapping – AWS Example**

### **Architecture**

- Chef Server on EC2
- Linux nodes migrated into AWS
- Chef manages packages, users, services

![Image](https://docs.chef.io/images/chef_overview_2020.svg)

![Image](https://docs.chef.io/images/server/server_components_14.svg)

### **Step-by-Step: Setup Chef Server on AWS**

1. Launch EC2 (Amazon Linux / Ubuntu)
2. Open ports:

   - 22 (SSH)
   - 443 (Chef)

3. Install Chef Server

```bash
curl -L https://omnitruck.chef.io/install.sh | sudo bash
sudo chef-server-ctl reconfigure
```

4. Create admin user and org
5. Download validation key

### **Bootstrap Client Node**

```bash
knife bootstrap <node-ip> \
--ssh-user ec2-user \
--sudo \
--node-name web01
```

## **5. Migration of Physical Servers to Cloud**

### **AWS Application Migration Service (MGN)**

MGN:

- Installs replication agent on physical server
- Replicates data continuously
- Launches EC2 from replicated image

## **Lab Assignments – Session 11 & 12**

## **Lab 1: Cloud API Integration for DC/DR Setup**

### **Objective**

- Automate EC2 provisioning using AWS APIs

### **Steps**

1. Configure AWS CLI

```bash
aws configure
```

2. Create EC2 using CLI
3. Attach EBS volume via API
4. Verify via console

### **Verification**

- EC2 visible in console
- Volume attached successfully

## **Lab 2: Automated Migration and Storage Synchronization**

### **Objective**

- Sync on-prem data to AWS

### **Steps**

1. Launch EC2 target
2. Install rsync
3. Run continuous sync job
4. Validate file integrity

### **Verification**

- Same files exist on EC2
- Checksums match

## **Lab 3: Configuring Chef / Puppet for Cloud Migration**

### **Objective**

- Enforce consistent configuration post-migration

### **Steps**

1. Setup Chef Server
2. Bootstrap migrated EC2
3. Apply cookbook to install Apache

```bash
sudo yum install httpd -y
```

### **Verification**

- Apache running on migrated server
- Config enforced via Chef

## **Lab 4: Physical to Cloud Server Migration**

### **Objective**

- Migrate physical Linux server to AWS

### **Steps**

1. Install MGN agent
2. Start replication
3. Launch test instance
4. Validate application

### **Verification**

- EC2 instance boots successfully
- Application accessible

## **Lab 5: Implementing High Availability**

### **Objective**

- Achieve HA using cloud APIs

### **Steps**

1. Create Auto Scaling Group
2. Add Load Balancer
3. Configure health checks

### **Verification**

- Instance auto-recreated on failure
- Application always reachable

# **Session 13 – Monitoring, Logging, and Auto-Healing**

_(2 Theory Hours + 4 Lab Hours)_

## **Theory Topics – Session 13**

## **1. Centralized Logging**

### **Why Centralized Logging**

- Troubleshooting
- Auditing
- Performance analysis

### **AWS Tools**

- CloudWatch Logs
- CloudWatch Agent
- S3 log archiving

## **2. Nagios**

### **What is Nagios**

- Traditional Network Monitoring System
- Host and service monitoring
- Alerting via email/SMS

## **3. Prometheus (Next-Gen NMS)**

### **Why Prometheus**

- Cloud-native
- Time-series based
- Pull-based monitoring
- Integrates with Kubernetes and AWS

## **4. Identifying Bottlenecks**

- CPU saturation
- Memory leaks
- Disk I/O wait
- Network latency

## **5. Auto-Scaling and Auto-Rebuilding**

AWS Auto Scaling:

- Scale based on CloudWatch metrics
- Replace unhealthy instances automatically

## **6. Updating Servers Without Downtime**

- Rolling updates
- Blue-Green deployments
- Load balancer based updates

## **7. Auto-Healing**

- Health checks
- Auto replacement
- Self-recovery

## **8. Cloud-Enabled Data Center Case Study**

### **Scenario**

- On-prem + AWS hybrid DC
- Auto-scaling web tier
- DR in another region
- Monitoring via Nagios + CloudWatch

## **Lab Assignments – Session 13**

## **Lab 6: Nagios on Linux – Monitor Linux Client**

### **Objective**

- Monitor Linux server health

### **Steps**

1. Launch EC2 for Nagios
2. Install Nagios Core
3. Configure host and services
4. Add Linux client with NRPE

### **Verification**

- Host shows UP
- CPU and disk metrics visible

## **Lab 7: Nagios on Linux – Monitor Windows Client**

### **Objective**

- Monitor Windows server from Linux Nagios

### **Steps**

1. Launch Windows EC2
2. Install NSClient++
3. Configure Nagios host definition
4. Enable firewall rules

### **Verification**

- Windows host visible in Nagios UI
- Services reporting status

# AWS SAA-C03 — Fast Professional Solution Reference

> **Purpose:** A fast review reference for AWS SAA-C03 questions.
>
> The goal is not to memorize answers. The goal is to recognize:
>
> `Requirement → Clue → AWS Capability → Best Solution → Eliminate Alternatives → Pattern`
>
> **Rule:** Do not explain the entire AWS service. Keep only the information needed to make the correct architectural decision.

---

## Q4 — Private Access to Amazon S3

### Problem

An EC2 instance in a VPC must access Amazon S3 **without Internet connectivity**.

### Answer

**Create a Gateway VPC Endpoint for Amazon S3.**

### Why

- **Primary Requirement:** Private network connectivity to S3.
- **Clue:** `EC2 + VPC + S3 + without Internet`
- S3 supports **Gateway VPC Endpoints**, allowing private access without an Internet Gateway or NAT Gateway.
- An IAM role provides authorization, but it does not create the network path.

### Solution Thinking

`Private VPC → S3 → Gateway VPC Endpoint`

### Why Not the Alternatives

- **CloudWatch Logs** → Does not provide private connectivity to S3.
- **API Gateway + PrivateLink** → Unnecessary architecture; S3 already supports a Gateway Endpoint.
- **EC2 Instance Profile** → Provides authorization, not network connectivity.

### Key Trap

> **IAM permissions ≠ Network connectivity**

### Quick Recognition

> **S3/DynamoDB + Private VPC access → Gateway VPC Endpoint**

### Final Takeaway

**Choose a Gateway VPC Endpoint because the primary requirement is private VPC connectivity to S3.**

---

## Q5 — Shared Files Across EC2 Instances

### Problem

Multiple EC2 instances in different Availability Zones must access the **same user files**.

### Answer

**Use Amazon EFS as the shared file system.**

### Why

- **Primary Requirement:** Shared file storage across multiple EC2 instances.
- **Clue:** `Multiple EC2 + Different AZs + Same Files`
- EFS provides a shared file system that multiple EC2 instances can access.
- EBS is not the appropriate shared file storage solution for this pattern.

### Solution Thinking

`Multiple EC2 → Shared Files → EFS`

### Why Not the Alternatives

- **Use one server for all files** → Creates unnecessary dependency on a single server.
- **Duplicate files across EBS volumes** → Requires synchronization and can cause consistency issues.
- **Load balancing** → Distributes traffic but does not synchronize storage.

### Key Trap

> **ALB distributes traffic; it does not synchronize storage.**

### Quick Recognition

> **Multiple EC2 + Shared Files → EFS**

### Final Takeaway

**Choose EFS because multiple EC2 instances need access to one shared file system.**

---

## Q6 — 70 TB Migration to S3

### Problem

A company must migrate **70 TB of on-premises data to S3 quickly while minimizing network bandwidth usage**.

### Answer

**Use AWS Snowball Edge.**

### Why

- **Primary Requirement:** Large-scale migration with minimal network usage.
- **Clue:** `70 TB + Fast Migration + Limited Bandwidth`
- Snowball Edge transfers data physically instead of sending the entire dataset over the network.
- AWS imports the transferred data into S3 after the device is returned.

### Solution Thinking

`Huge Data + Limited Bandwidth → Physical Transfer → Snowball Edge`

### Why Not the Alternatives

- **Direct Connect** → Improves connectivity but still transfers all data over the network.
- **AWS CLI to S3** → Requires transferring the entire 70 TB through the network.
- **S3 File Gateway** → Still depends on network connectivity for data transfer.

### Key Trap

> **Direct Connect improves connectivity; Snowball minimizes network transfer.**

### Quick Recognition

> **Huge Data + Limited Bandwidth → Snowball Edge**

### Final Takeaway

**Choose Snowball Edge because physical transfer minimizes network bandwidth usage for massive datasets.**

---

## Q7 — Many Consumers and Message Spikes

### Problem

One application produces messages that must be consumed by dozens of applications, with sudden traffic spikes.

### Answer

**Publish messages to an SNS topic with multiple SQS subscriptions.**

### Why

- **Primary Requirement:** Fan-out with independent consumer processing.
- **Clue:** `One Producer + Many Consumers + Traffic Spikes`
- SNS provides **fan-out** to multiple consumers.
- Each consumer can have its own SQS queue for independent buffering and processing.

### Solution Thinking

`Producer → SNS → Multiple SQS Queues → Consumers`

### Why Not the Alternatives

- **EC2 Auto Scaling** → Scales compute but does not provide message fan-out.
- **Kinesis Data Analytics** → Designed for stream analytics, not this queue-based fan-out pattern.
- **Single Kinesis shard** → Does not match the required scalable fan-out architecture.

### Key Trap

> **SNS = Fan-out | SQS = Queue / Buffer**

### Quick Recognition

> **One Producer + Many Consumers → SNS + Multiple SQS Queues**

### Final Takeaway

**Choose SNS + SQS because the primary requirement is scalable fan-out with independent consumer queues.**

---

## Q8 — Decoupled Jobs with Auto Scaling

### Problem

Jobs are processed by compute workers, workloads are variable, and the architecture must become more resilient and scalable.

### Answer

**Use Amazon SQS for jobs and an EC2 Auto Scaling group that scales based on queue depth.**

### Why

- **Primary Requirement:** Decouple jobs from workers and scale workers dynamically.
- **Clue:** `Variable Jobs + Workers + Scalability`
- SQS provides a durable buffer between producers and workers.
- Queue depth directly represents pending work and can drive worker scaling.

### Solution Thinking

`Jobs → SQS → EC2 Workers → Scale by Queue Depth`

### Why Not the Alternatives

- **Scheduled Scaling** → Best for predictable workloads, not variable demand.
- **CloudTrail** → Used for auditing, not job processing.
- **EventBridge** → Routes events but does not provide the required durable job queue.

### Key Trap

> **Scale workers based on pending work, not only CPU utilization.**

### Quick Recognition

> **Variable Jobs + Workers → SQS + ASG**

### Final Takeaway

**Choose SQS + Auto Scaling because it decouples jobs and scales workers according to workload.**

---

## Q9 — On-Premises SMB Storage Extension

### Problem

An on-premises SMB file server is running out of capacity. Recent files need accessible storage, while older files should be archived automatically.

### Answer

**Use Amazon S3 File Gateway with an S3 Lifecycle policy.**

### Why

- **Primary Requirement:** Extend existing SMB storage into AWS.
- **Clue:** `On-Prem SMB + S3 + Automatic Archive`
- S3 File Gateway provides SMB access backed by Amazon S3.
- S3 Lifecycle policies automatically transition older objects to archival storage.

### Solution Thinking

`On-Prem SMB → S3 File Gateway → S3 → Lifecycle → Archive`

### Why Not the Alternatives

- **FSx for Windows File Server** → Provides SMB storage but does not match the S3 lifecycle/archive requirement.
- **S3 client utility** → Requires application or client changes.
- **DataSync** → Moves or synchronizes data; it is not an ongoing file-access solution.

### Key Trap

> **DataSync = Data movement | File Gateway = Ongoing file access**

### Quick Recognition

> **SMB/NFS + S3 Storage → S3 File Gateway**

### Final Takeaway

**Choose S3 File Gateway because it extends existing SMB access into S3 with lifecycle-based archiving.**

---

## Q10 — Read-Heavy MySQL Database

### Problem

A MySQL workload is **read-heavy**, read demand is unpredictable, and the database requires high availability.

### Answer

**Use Amazon Aurora with Aurora Replicas and Aurora Auto Scaling.**

### Why

- **Primary Requirement:** Automatically scale database read capacity.
- **Clue:** `MySQL + Read Heavy + Unpredictable Reads + HA`
- Aurora Replicas provide read scaling.
- Aurora Auto Scaling adjusts replica capacity based on demand.
- A Multi-AZ architecture supports high availability.

### Solution Thinking

`MySQL → Aurora → Read Replicas → Auto Scaling`

### Why Not the Alternatives

- **Single-AZ RDS** → Does not meet the high-availability requirement.
- **ElastiCache** → Can reduce database reads but does not provide database replica scaling.
- **Redshift** → Designed for analytics and data warehousing, not transactional workloads.

### Key Trap

> **Multi-AZ = Availability | Read Replicas = Read Scaling**

### Quick Recognition

> **Read-Heavy + Unpredictable Reads → Aurora Replicas + Auto Scaling**

### Final Takeaway

**Choose Aurora Replicas with Auto Scaling because the primary requirement is scalable read capacity.**

---

## Q53 — HTTP to HTTPS Redirect

### Problem

An ALB receives both HTTP and HTTPS traffic, and all HTTP requests must automatically use HTTPS.

### Answer

**Create an ALB listener rule to redirect HTTP traffic to HTTPS.**

### Why

- **Primary Requirement:** Automatic HTTP-to-HTTPS redirection.
- **Clue:** `ALB + HTTP + HTTPS + Redirect`
- ALB listener rules support **Redirect actions**.
- The HTTP listener can redirect requests directly to HTTPS.

### Solution Thinking

`HTTP → ALB Listener → Redirect → HTTPS`

### Why Not the Alternatives

- **Network ACL** → Filters network traffic; it does not redirect URLs.
- **Network Load Balancer + SNI** → SNI is related to TLS, not HTTP-to-HTTPS redirection.
- **Custom solution** → Unnecessary when ALB provides native redirect support.

### Key Trap

> **HTTP → HTTPS redirect = ALB Listener Redirect**

### Quick Recognition

> **ALB + HTTP → HTTPS → Listener Redirect**

### Final Takeaway

**Choose an ALB listener redirect because the requirement is automatic HTTP-to-HTTPS redirection.**

---

## Q54 — Automatic Database Credential Rotation

### Problem

An EC2 application needs RDS credentials without hardcoding them, with **automatic credential rotation**.

### Answer

**Use AWS Secrets Manager with automatic rotation and an EC2 IAM role.**

### Why

- **Primary Requirement:** Secure secret storage with automatic rotation.
- **Clue:** `Database Credentials + Automatic Rotation`
- Secrets Manager is designed to store and manage credentials.
- It supports automatic rotation for supported secrets.
- The EC2 IAM role retrieves the secret without embedding credentials in application code.

### Solution Thinking

`RDS Credentials → Secrets Manager → Automatic Rotation → EC2 IAM Role`

### Why Not the Alternatives

- **S3 + Lambda** → Requires custom secret storage and rotation logic.
- **Instance Metadata** → Not a secret-management service.
- **Parameter Store** → Can store secrets, but the native automatic rotation requirement points to Secrets Manager.

### Key Trap

> **Secret storage + automatic rotation → Secrets Manager**

### Quick Recognition

> **DB Credentials + Automatic Rotation → Secrets Manager**

### Final Takeaway

**Choose Secrets Manager because it natively manages and automatically rotates database credentials.**

---

## Q55 — External CA SSL/TLS Certificate

### Problem

An ALB needs an SSL/TLS certificate issued by an **external certificate authority**.

### Answer

**Import the external certificate into AWS Certificate Manager (ACM) and attach it to the ALB.**

### Why

- **Primary Requirement:** Use a certificate issued by an external CA.
- **Clue:** `External CA`
- ACM supports importing third-party certificates for use with supported AWS services.
- Imported certificates do not receive ACM-managed renewal, so expiration must be monitored and renewal handled externally.

### Solution Thinking

`External CA → Import into ACM → ALB → Manual Renewal`

### Why Not the Alternatives

- **ACM-issued certificate** → The certificate is issued by ACM, not the external CA.
- **ACM Private CA** → Used to issue private certificates.
- **Managed renewal** → Does not apply to imported certificates.

### Key Trap

> **External CA ≠ ACM-issued certificate**

> **Imported certificate ≠ ACM Managed Renewal**

### Quick Recognition

> **External CA Certificate → Import into ACM**

### Final Takeaway

**Choose ACM import because the certificate must originate from an external certificate authority.**

---

## Q56 — Scalable PDF to JPG Processing

### Problem

PDF files must be uploaded, automatically converted to JPG, and processed with scalability and minimal operational overhead.

### Answer

**Store files in S3 and use an S3 event to invoke Lambda for conversion.**

### Why

- **Primary Requirement:** Scalable event-driven file processing.
- **Clue:** `File Upload + Automatic Processing + Scale`
- S3 stores both original and processed files.
- S3 Event Notifications invoke Lambda automatically.
- Lambda provides serverless scaling without managing EC2 instances.

### Solution Thinking

`PDF Upload → S3 → Event → Lambda → JPG → S3`

### Why Not the Alternatives

- **EC2 + EBS** → Requires server management.
- **DynamoDB** → Not designed for storing large PDF/JPG objects.
- **EC2 + EFS** → Technically possible but adds unnecessary infrastructure.

### Key Trap

> **File/Object Storage → S3**

> **Event-Driven Processing → S3 Event + Lambda**

### Quick Recognition

> **File Upload + Automatic Processing → S3 + Lambda**

### Final Takeaway

**Choose S3 + Lambda because it provides scalable event-driven processing with minimal operational overhead.**

---

## Q57 — Identify PHI in Medical Documents

### Problem

PDF and JPEG medical documents require text extraction followed by identification of **Protected Health Information (PHI)**.

### Answer

**Use Amazon Textract to extract text and Amazon Comprehend Medical to identify PHI.**

### Why

- **Primary Requirement:** Extract document text and identify medical sensitive information.
- **Textract** extracts text from documents and images.
- **Comprehend Medical** identifies medical entities and PHI.
- Managed AI services avoid building and maintaining custom OCR and ML solutions.

### Solution Thinking

`Medical Document → Textract → Extracted Text → Comprehend Medical → PHI`

### Why Not the Alternatives

- **Rekognition** → Designed primarily for image and video analysis, not document OCR.
- **Python libraries** → Require custom implementation and maintenance.
- **SageMaker** → Unnecessary custom ML when managed specialized services exist.

### Key Trap

> **Text Extraction ≠ PHI Detection**

### Quick Recognition

> **Medical Document → Textract → Comprehend Medical**

### Final Takeaway

**Choose Textract + Comprehend Medical because each service directly handles one required processing step.**

---

## Q58 — SQS Duplicate Processing

### Problem

EC2 workers process SQS messages and write to RDS, but duplicate records occur when processing takes too long.

### Answer

**Increase the SQS visibility timeout.**

### Why

- **Primary Requirement:** Prevent the same message from becoming available before processing finishes.
- A received message becomes invisible for the visibility timeout period.
- If processing exceeds that timeout, the message can become visible again.
- Another worker can then process the same message before the first worker deletes it.

### Solution Thinking

`SQS → Processing → RDS → Delete`

`Processing Time > Visibility Timeout → Duplicate Processing`

### Why Not the Alternatives

- **CreateQueue** → Creating another queue does not solve the timeout issue.
- **AddPermission** → Permissions do not prevent duplicate processing.
- **Long Polling** → Reduces empty receives, not duplicate processing.

### Key Trap

> **SQS Standard = At-Least-Once Delivery**

### Quick Recognition

> **SQS Duplicate + Long Processing → Increase Visibility Timeout**

### Final Takeaway

**Choose a longer visibility timeout because the message is becoming visible again before processing finishes.**

---

## Q59 — Direct Connect with Low-Cost Backup

### Problem

A hybrid environment requires **consistent low latency** for the primary connection, while the backup connection can be slower and should minimize cost.

### Answer

**Use AWS Direct Connect as the primary connection and a Site-to-Site VPN as the backup.**

### Why

- **Primary Requirement:** Consistent low-latency primary connectivity.
- Direct Connect provides dedicated connectivity.
- VPN provides a lower-cost backup path.
- The backup is allowed to be slower, making DX + VPN the best fit.

### Solution Thinking

`On-Premises → Direct Connect (Primary)`

`On-Premises → VPN (Backup)`

### Why Not the Alternatives

- **Two VPN connections** → Do not satisfy the consistent low-latency primary requirement.
- **Two Direct Connect connections** → Higher availability but unnecessary cost.
- **Direct Connect only** → Does not provide the required backup connection.

### Key Trap

> **High-performance primary + low-cost backup → Direct Connect + VPN**

### Quick Recognition

> **Consistent Low Latency + Cheaper Backup → Direct Connect + VPN**

### Final Takeaway

**Choose Direct Connect + VPN because DX meets the primary performance requirement while VPN provides a lower-cost backup.**

---

## Q60 — Highly Available EC2 and Database Architecture

### Problem

A business-critical application and database run in a single Availability Zone. The company requires high availability and minimal downtime.

### Answer

**Deploy the application across multiple AZs and use a Multi-AZ database configuration.**

### Why

- **Primary Requirement:** Eliminate a single-AZ failure as a single point of failure.
- EC2 instances across multiple AZs improve application availability.
- A Multi-AZ database configuration provides failover capability.
- Managed HA features require less operational effort than a custom DR design.

### Solution Thinking

`ALB → Auto Scaling Across AZs → Multi-AZ Database`

### Why Not the Alternatives

- **Cross-Region architecture** → More complex than required for in-Region high availability.
- **Single AZ + snapshots** → Does not prevent downtime during an AZ failure.
- **Custom S3 + Lambda recovery** → Adds unnecessary complexity.

### Key Trap

> **Multi-AZ = High Availability**

> **Cross-Region = Disaster Recovery**

### Quick Recognition

> **Business-Critical + Minimum Downtime → Multi-AZ**

### Final Takeaway

**Choose Multi-AZ because the primary requirement is high availability within the AWS Region.**

---

## Q61 — DynamoDB RPO of 15 Minutes

### Problem

DynamoDB data can become corrupted, and the company needs recovery to a recent point in time.

### Answer

**Enable DynamoDB Point-in-Time Recovery (PITR).**

### Why

- **Primary Requirement:** Restore data to a specific recent point in time.
- PITR continuously maintains recovery points.
- The table can be restored to a selected point within the supported recovery window.
- It is more appropriate than infrequent backups when the required RPO is small.

### Solution Thinking

`DynamoDB → PITR → Data Corruption → Point-in-Time Restore`

### Why Not the Alternatives

- **Global Tables** → Provide replication and availability, not point-in-time recovery.
- **Daily S3 backups** → Cannot meet a 15-minute RPO.
- **EBS snapshots** → Not a DynamoDB backup mechanism.

### Key Trap

> **Replication/HA ≠ Point-in-Time Recovery**

### Quick Recognition

> **DynamoDB + Corruption + Specific Recovery Time → PITR**

### Final Takeaway

**Choose DynamoDB PITR because the requirement is recovery to a recent point in time after corruption.**

---

## Q62 — DynamoDB 7-Year Retention

### Problem

DynamoDB transaction data must be retained for **7 years** with minimal operational effort.

### Answer

**Use AWS Backup with backup schedules and retention policies.**

### Why

- **Primary Requirement:** Long-term managed backup retention.
- AWS Backup provides centralized backup scheduling.
- Retention policies manage how long backups are retained.
- No custom Lambda, EventBridge, or manual backup process is required.

### Solution Thinking

`DynamoDB → AWS Backup → Scheduled Backups → Long-Term Retention`

### Why Not the Alternatives

- **PITR** → Designed primarily for point-in-time recovery, not the best match for long-term scheduled retention.
- **Manual backup + S3 Lifecycle** → Requires manual management and additional architecture.
- **EventBridge + Lambda + S3** → Custom automation creates higher operational overhead.

### Key Trap

> **PITR = Point-in-Time Recovery**

> **AWS Backup = Managed Backup + Retention**

### Quick Recognition

> **Long-Term Backup Retention → AWS Backup**

### Final Takeaway

**Choose AWS Backup because the primary requirement is long-term managed backup retention.**

---

## Q63 — EC2 Purchasing for Different Environments

### Problem

Production EC2 instances run **24/7**, while development and test instances run only during working hours and are stopped when not in use.

### Answer

**Use Reserved Instances for production and On-Demand Instances for development and test.**

### Why

- **Production:** Predictable continuous usage makes Reserved Instances cost-effective.
- **Development/Test:** Intermittent usage requires flexibility.
- Stopping Dev/Test instances reduces usage hours, making long-term commitment less suitable.

### Solution Thinking

`Production 24/7 → Reserved`

`Dev/Test Intermittent → On-Demand`

### Why Not the Alternatives

- **Spot for Production** → Interruption risk is unsuitable for this production workload.
- **Reserved for Dev/Test** → Usage is not continuous or predictable enough.
- **Spot Blocks** → Not the appropriate purchasing model for this usage pattern.

### Key Trap

> **Predictable Continuous Usage → Reserved**

> **Intermittent Usage → On-Demand**

### Quick Recognition

> **Production 24/7 + Dev/Test Scheduled → Reserved + On-Demand**

### Final Takeaway

**Choose Reserved for continuously running production and On-Demand for intermittent Dev/Test workloads.**

---

## Q64 — Immutable S3 Documents

### Problem

A regulatory requirement states that documents cannot be **modified or deleted** after storage.

### Answer

**Enable S3 Versioning and S3 Object Lock.**

### Why

- **Primary Requirement:** Immutable/WORM storage.
- S3 Object Lock prevents objects from being modified or deleted during the retention period.
- Versioning is required for Object Lock.
- This directly matches the regulatory immutability requirement.

### Solution Thinking

`Documents → S3 Versioning + Object Lock → WORM / Immutable`

### Why Not the Alternatives

- **Lifecycle Policy** → Manages storage transitions, not immutability.
- **Versioning + Read-Only ACL** → Access control does not provide WORM protection.
- **EFS Read-Only Mount** → Does not guarantee immutable storage.

### Key Trap

> **Read-Only Access ≠ WORM Protection**

### Quick Recognition

> **Regulatory + Cannot Modify/Delete → S3 Object Lock**

### Final Takeaway

**Choose S3 Object Lock because the primary requirement is immutable/WORM document storage.**

---

## Q65 — Protect S3 from Accidental Deletion

### Problem

Important audit documents in S3 require stronger protection against **accidental deletion**.

### Answer

**Enable S3 Versioning and MFA Delete.**

### Why

- Versioning preserves previous object versions after deletion or overwrite.
- MFA Delete adds an additional authentication requirement for sensitive permanent deletion operations.
- The solution directly targets accidental deletion risk.

### Solution Thinking

`Important S3 Data → Versioning + MFA Delete → Deletion Protection`

### Why Not the Alternatives

- **MFA on IAM users** → Improves authentication but does not provide S3-specific deletion protection.
- **Lifecycle Policy** → Does not prevent accidental deletion.
- **KMS Encryption** → Protects confidentiality, not deletion.

### Key Trap

> **Encryption protects confidentiality ≠ Deletion protection**

### Quick Recognition

> **S3 + Accidental Deletion → Versioning + MFA Delete**

### Final Takeaway

**Choose Versioning + MFA Delete because the primary requirement is protection against accidental permanent deletion.**

---

# Pattern Recognition

```text
Private VPC + S3
→ Gateway VPC Endpoint

Private VPC + DynamoDB
→ Gateway VPC Endpoint

Multiple EC2 + Shared Files
→ EFS

Huge Data + Limited Bandwidth
→ Snowball Edge

One Producer + Many Consumers
→ SNS + Multiple SQS Queues

Variable Jobs + Worker Scaling
→ SQS + Auto Scaling Group

On-Prem SMB/NFS + S3
→ S3 File Gateway

Read-Heavy Database
→ Read Replicas

Unpredictable Read Demand
→ Auto Scaling

ALB + HTTP → HTTPS
→ Listener Redirect

DB Credentials + Automatic Rotation
→ Secrets Manager

External CA Certificate
→ Import into ACM

File Upload + Automatic Processing
→ S3 Event + Lambda

Medical Documents + PHI
→ Textract + Comprehend Medical

SQS Duplicate Processing
→ Increase Visibility Timeout

Consistent Low Latency + Low-Cost Backup
→ Direct Connect + VPN

High Availability Within One Region
→ Multi-AZ

DynamoDB Corruption + Specific Recovery Point
→ PITR

Long-Term Backup Retention
→ AWS Backup

EC2 24/7 + Predictable Usage
→ Reserved Instances

EC2 Intermittent Usage
→ On-Demand

Immutable / WORM S3 Data
→ Object Lock + Versioning

S3 Accidental Deletion Protection
→ Versioning + MFA Delete

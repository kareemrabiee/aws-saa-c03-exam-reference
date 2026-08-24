# AWS SAA-C03 — Fast Professional Solution Reference

> **Purpose:** مراجعة سريعة لأسئلة AWS SAA-C03.
>
> الهدف ليس حفظ الإجابة، بل معرفة:
>
> `Requirement → Clue → AWS Capability → Best Solution → Eliminate Alternatives → Pattern`
>
> **Rule:** لا تشرح Service بالكامل. احتفظ فقط بالمعلومة التي تساعد في حل سؤال مشابه لاحقًا.

---

# Q4 — Private Access to Amazon S3

### Problem

EC2 داخل VPC يحتاج الوصول إلى S3 **بدون Internet connectivity**.

### Answer

**Create a Gateway VPC Endpoint for Amazon S3.**

### Why

- **Primary Requirement:** Private network connectivity to S3.
- **Clue:** `EC2 + VPC + S3 + without Internet`
- S3 يدعم **Gateway VPC Endpoint** للوصول الخاص بدون Internet Gateway أو NAT Gateway.
- IAM Role يحدد authorization، لكنه لا ينشئ network path إلى S3.

### Solution Thinking

`Private VPC → S3 → Gateway VPC Endpoint`

### Why Not the Alternatives

- **CloudWatch Logs** → لا يوفر private connectivity إلى S3.
- **API Gateway + PrivateLink** → Architecture غير ضرورية؛ S3 لديه Gateway Endpoint.
- **EC2 Instance Profile** → يوفر authorization فقط، وليس network connectivity.

### Key Trap

> **IAM permissions ≠ Network connectivity**

### Quick Recognition

> **S3/DynamoDB + Private VPC access → Gateway VPC Endpoint**

### Final Takeaway

**Choose a Gateway VPC Endpoint because the primary requirement is private VPC connectivity to S3.**

---

# Q5 — Shared Files Across EC2 Instances

### Problem

عدة EC2 instances في AZs مختلفة تحتاج الوصول إلى **نفس ملفات المستخدمين**.

### Answer

**Use Amazon EFS as the shared file system.**

### Why

- **Primary Requirement:** Shared file storage across multiple EC2 instances.
- **Clue:** `Multiple EC2 + Different AZs + Same Files`
- EFS يوفر file system مشتركًا يمكن لعدة EC2 instances الوصول إليه.
- EBS volume ليس shared file system مناسبًا لهذا pattern.

### Solution Thinking

`Multiple EC2 → Shared Files → EFS`

### Why Not the Alternatives

- **One server handles all files** → يخلق dependency على server واحد.
- **Duplicate files across EBS** → يحتاج synchronization ويخلق consistency problems.
- **Load balancing** → يوزع requests، لكنه لا يوحّد storage.

### Key Trap

> **ALB distributes traffic; it does not synchronize storage.**

### Quick Recognition

> **Multiple EC2 + Shared Files → EFS**

### Final Takeaway

**Choose EFS because multiple EC2 instances need shared file storage.**

---

# Q6 — 70 TB Migration to S3

### Problem

نقل **70 TB** من on-premises إلى S3 بسرعة مع تقليل استخدام network bandwidth.

### Answer

**Use AWS Snowball Edge.**

### Why

- **Primary Requirement:** Large-scale migration with minimal network usage.
- **Clue:** `70 TB + Fast Migration + Limited Bandwidth`
- Snowball Edge ينقل البيانات physical بدل إرسال كل البيانات عبر network.
- بعد إعادة الجهاز، AWS ينقل البيانات إلى S3.

### Solution Thinking

`Huge Data + Limited Bandwidth → Physical Transfer → Snowball Edge`

### Why Not the Alternatives

- **Direct Connect** → يحسن الاتصال لكنه لا يلغي network transfer.
- **AWS CLI → S3** → ينقل الـ70 TB عبر network.
- **S3 File Gateway** → يعتمد أيضًا على network connectivity.

### Key Trap

> **Direct Connect improves connectivity; Snowball minimizes network transfer.**

### Quick Recognition

> **Huge Data + Limited Bandwidth → Snowball Edge**

### Final Takeaway

**Choose Snowball Edge because physical transfer minimizes network bandwidth usage.**

---

# Q7 — Many Consumers and Message Spikes

### Problem

Application تنتج messages تستهلكها عشرات التطبيقات، مع spikes كبيرة في traffic.

### Answer

**Publish to SNS and use multiple SQS subscriptions.**

### Why

- **Primary Requirement:** Fan-out + independent consumer processing.
- **Clue:** `One Producer + Many Consumers + Spikes`
- SNS يوفر **fan-out**.
- كل consumer يحصل على SQS queue مستقلة للـ buffering والـ processing.

### Solution Thinking

`Producer → SNS → Multiple SQS Queues → Consumers`

### Why Not the Alternatives

- **EC2 Auto Scaling** → scales compute، لكنه لا يوفر message fan-out.
- **Kinesis Analytics** → مخصص أكثر لـ streaming/analytics.
- **Single Kinesis shard** → لا يطابق هذا الـ fan-out queue pattern.

### Key Trap

> **SNS = Fan-out | SQS = Queue / Buffer**

### Quick Recognition

> **One Producer + Many Consumers → SNS + Multiple SQS**

### Final Takeaway

**Choose SNS + SQS because the primary requirement is scalable fan-out with independent consumer queues.**

---

# Q8 — Decoupled Jobs with Auto Scaling

### Problem

Jobs موزعة على compute workers، والـ workload variable. المطلوب resilience وdynamic scaling.

### Answer

**Use SQS for jobs and an EC2 Auto Scaling Group based on queue depth.**

### Why

- **Primary Requirement:** Decouple jobs from workers and scale workers dynamically.
- **Clue:** `Variable Jobs + Workers + Scaling`
- SQS يعمل كـ durable buffer.
- Queue depth تمثل backlog الحقيقي، ويمكن استخدامها لتحديد عدد workers.

### Solution Thinking

`Jobs → SQS → EC2 Workers → Scale by Queue Depth`

### Why Not the Alternatives

- **Scheduled Scaling** → مناسب أكثر للـ predictable workloads.
- **CloudTrail** → auditing وليس job processing.
- **EventBridge** → event routing، وليس durable work queue.

### Key Trap

> **Scale workers according to pending work, not only CPU.**

### Quick Recognition

> **Variable Jobs + Workers → SQS + ASG**

### Final Takeaway

**Choose SQS + ASG because it decouples jobs and scales workers based on workload.**

---

# Q9 — On-Premises SMB Storage Extension

### Problem

SMB storage on-premises يحتاج expansion، مع الاحتفاظ بالـ recent files وإمكانية archive للملفات القديمة.

### Answer

**Use S3 File Gateway with S3 Lifecycle policies.**

### Why

- **Primary Requirement:** Extend existing SMB storage into AWS.
- **Clue:** `On-Prem SMB + S3 + Archive`
- S3 File Gateway يوفر SMB access backed by S3.
- S3 Lifecycle ينقل objects القديمة إلى archival storage.

### Solution Thinking

`On-Prem SMB → S3 File Gateway → S3 → Lifecycle → Archive`

### Why Not the Alternatives

- **FSx for Windows File Server** → managed SMB، لكنه لا يطابق S3 lifecycle/archive pattern.
- **S3 Client Utility** → يحتاج application/client changes.
- **DataSync** → data movement/synchronization وليس ongoing file access.

### Key Trap

> **DataSync = Data movement | File Gateway = Ongoing file access**

### Quick Recognition

> **SMB/NFS + S3 → S3 File Gateway**

### Final Takeaway

**Choose S3 File Gateway because it extends SMB access into scalable S3 storage.**

---

# Q10 — Read-Heavy MySQL Database

### Problem

MySQL workload **read-heavy**، والـ read demand unpredictable، مع الحاجة إلى high availability.

### Answer

**Use Aurora with Aurora Replicas and Auto Scaling across AZs.**

### Why

- **Primary Requirement:** Scalable read capacity.
- **Clue:** `MySQL + Read Heavy + Unpredictable Reads + HA`
- Aurora Replicas توفر read scaling.
- Aurora Auto Scaling يضيف/removes replicas حسب demand.
- Multi-AZ architecture تدعم high availability.

### Solution Thinking

`MySQL → Aurora → Read Replicas → Auto Scaling`

### Why Not the Alternatives

- **Single-AZ RDS** → لا يحقق HA.
- **ElastiCache** → caching وليس database read-replica scaling.
- **Redshift** → analytics/data warehouse وليس transactional workload.

### Key Trap

> **Multi-AZ = Availability | Read Replicas = Read Scaling**

### Quick Recognition

> **Read-heavy + unpredictable reads → Aurora Replicas + Auto Scaling**

### Final Takeaway

**Choose Aurora Replicas with Auto Scaling because the primary requirement is scalable read capacity.**

---

# Q53 — HTTP to HTTPS Redirect

### Problem

ALB يستقبل HTTP وHTTPS، والمطلوب إجبار clients على استخدام HTTPS.

### Answer

**Create an ALB listener rule to redirect HTTP to HTTPS.**

### Why

- ALB Listener Rules تدعم **Redirect actions**.
- HTTP listener يستطيع إعادة توجيه request إلى HTTPS.
- الحل مباشر ولا يحتاج network architecture إضافية.

### Solution Thinking

`HTTP → ALB Listener → Redirect → HTTPS`

### Why Not the Alternatives

- **Network ACL** → network filtering وليس URL redirect.
- **NLB + SNI** → SNI متعلق بـ TLS وليس redirect.
- **Custom URL rule** → unnecessary.

### Key Trap

> **HTTP → HTTPS redirect = ALB Listener Redirect**

### Quick Recognition

> **ALB + HTTP → HTTPS → Listener Redirect**

### Final Takeaway

**Choose an ALB listener redirect because the requirement is automatic HTTP-to-HTTPS redirection.**

---

# Q54 — Automatic Database Credential Rotation

### Problem

EC2 application تحتاج RDS credentials بدون hardcoding، مع **automatic rotation**.

### Answer

**Use AWS Secrets Manager with automatic rotation.**

### Why

- **Primary Requirement:** Secure secret storage + automatic rotation.
- **Clue:** `DB Credentials + Automatic Rotation`
- Secrets Manager مصمم لإدارة secrets مثل database credentials.
- EC2 يستخدم IAM Role للوصول إلى secret بدون وضع credentials في code.

### Solution Thinking

`RDS Credentials → Secrets Manager → Automatic Rotation → EC2`

### Why Not the Alternatives

- **S3 + Lambda** → custom implementation وoperational overhead أعلى.
- **Instance Metadata** → ليست secret-management service.
- **Parameter Store** → لا يطابق native automatic rotation requirement بنفس الشكل.

### Key Trap

> **Secret storage + automatic rotation → Secrets Manager**

### Quick Recognition

> **DB credentials + automatic rotation → Secrets Manager**

### Final Takeaway

**Choose Secrets Manager because it natively manages and rotates database credentials.**

---

# Q55 — External CA SSL/TLS Certificate

### Problem

ALB يحتاج certificate صادر من **external CA** مع الحاجة لإدارته داخل AWS.

### Answer

**Import the external certificate into ACM and attach it to the ALB.**

### Why

- **Primary Requirement:** Use a certificate issued by an external CA.
- **Clue:** `External CA`
- ACM يسمح بـ **imported certificates** واستخدامها مع ALB.
- Imported certificates لا تحصل على ACM-managed renewal، لذلك يجب مراقبة expiration وتجديدها خارجيًا.

### Solution Thinking

`External CA → Import to ACM → ALB → Manual Renewal`

### Why Not the Alternatives

- **ACM-issued certificate** → certificate سيكون صادرًا من ACM وليس external CA.
- **ACM Private CA** → يستخدم لإصدار private certificates.
- **Managed Renewal** → لا ينطبق على imported certificates.

### Key Trap

> **External CA ≠ ACM-issued certificate**

> **Imported ACM certificate ≠ ACM Managed Renewal**

### Quick Recognition

> **External CA → Import into ACM**

### Final Takeaway

**Choose ACM import because the certificate must originate from an external CA.**

---

# Q56 — Scalable PDF to JPG Processing

### Problem

PDF files يتم رفعها، ويجب تحويلها تلقائيًا إلى JPG مع scalability وأقل operational overhead.

### Answer

**Store PDFs in S3 and trigger Lambda through an S3 event.**

### Why

- **Primary Requirement:** Event-driven scalable file processing.
- **Clue:** `File Upload + Automatic Processing + Scale`
- S3 يخزن original وprocessed objects.
- S3 Event Notification تشغل Lambda عند upload.
- Lambda serverless وتتوسع بدون إدارة servers.

### Solution Thinking

`PDF → S3 → Event → Lambda → JPG → S3`

### Why Not the Alternatives

- **EC2 + EBS** → يحتاج server management.
- **DynamoDB** → ليس object storage للملفات الكبيرة.
- **EC2 + EFS** → ممكن، لكنه architecture أكثر تعقيدًا.

### Key Trap

> **File/Object storage → S3**

> **Event-driven processing → S3 Event + Lambda**

### Quick Recognition

> **Upload + automatic processing → S3 + Lambda**

### Final Takeaway

**Choose S3 + Lambda because it provides scalable event-driven processing with minimal operations.**

---

# Q57 — Identify PHI in Medical Documents

### Problem

Medical PDF/JPEG documents تحتاج استخراج النص وتحديد **Protected Health Information (PHI)**.

### Answer

**Use Amazon Textract followed by Amazon Comprehend Medical.**

### Why

- **Textract** → extracts text from documents/images.
- **Comprehend Medical** → identifies medical entities and PHI.
- Managed services تقلل الحاجة إلى custom OCR/ML infrastructure.

### Solution Thinking

`Medical Document → Textract → Text → Comprehend Medical → PHI`

### Why Not the Alternatives

- **Rekognition** → image/video analysis وليس document text extraction.
- **Python OCR libraries** → custom maintenance.
- **SageMaker** → unnecessary custom ML عندما توجد managed services مناسبة.

### Key Trap

> **Text Extraction ≠ PHI Detection**

### Quick Recognition

> **Medical document → Textract → Comprehend Medical**

### Final Takeaway

**Choose Textract + Comprehend Medical because each service directly handles one required processing step.**

---

# Q58 — SQS Duplicate Processing

### Problem

EC2 workers تقرأ SQS وتكتب إلى RDS، لكن تظهر duplicate records لأن processing أحيانًا يستغرق وقتًا طويلًا.

### Answer

**Increase the SQS visibility timeout.**

### Why

- بعد ReceiveMessage تصبح الرسالة invisible لفترة محددة.
- إذا انتهت الـ visibility timeout قبل انتهاء processing، تصبح الرسالة visible مرة أخرى.
- Worker آخر قد يعالج نفس الرسالة قبل حذفها.

### Solution Thinking

`SQS → Processing → RDS → Delete`

`Processing Time > Visibility Timeout → Duplicate Processing`

### Why Not the Alternatives

- **CreateQueue** → لا يعالج timeout.
- **AddPermission** → authorization وليس duplicate handling.
- **Long Polling** → يقلل empty receives، لكنه لا يحل visibility timeout.

### Key Trap

> **SQS Standard → At-Least-Once Delivery**

### Quick Recognition

> **SQS duplicate + long processing → Increase Visibility Timeout**

### Final Takeaway

**Choose a longer visibility timeout because the message is becoming visible again before processing finishes.**

---

# Q59 — Direct Connect with Low-Cost Backup

### Problem

Hybrid connection تحتاج **consistent low latency** مع backup أقل تكلفة ويمكن أن يكون أبطأ.

### Answer

**Use Direct Connect as primary and VPN as backup.**

### Why

- **Primary Requirement:** Consistent low-latency connectivity.
- Direct Connect يوفر dedicated network connection.
- VPN يوفر lower-cost backup.
- قبول slower backup يجعل DX + VPN أفضل من two DX connections من ناحية التكلفة.

### Solution Thinking

`On-Prem → Direct Connect (Primary)`

`On-Prem → VPN (Backup)`

### Why Not the Alternatives

- **Two VPN tunnels** → لا يحقق low-latency primary requirement.
- **Two Direct Connect connections** → HA أعلى لكن cost أعلى.
- **Automatic DX-only solution** → لا يحقق low-cost backup requirement.

### Key Trap

> **High-performance primary + low-cost backup → DX + VPN**

### Quick Recognition

> **Consistent low latency + cheaper backup → Direct Connect + VPN**

### Final Takeaway

**Choose Direct Connect + VPN because DX provides the primary performance while VPN provides a cheaper backup.**

---

# Q60 — Highly Available EC2 + Aurora

### Problem

Business-critical application وdatabase يعملان في single AZ. المطلوب high availability وminimum downtime.

### Answer

**Use Multi-AZ compute and database architecture.**

### Why

- **Primary Requirement:** Eliminate single-AZ failure.
- EC2 instances عبر multiple AZs تمنع application outage بسبب AZ failure.
- Multi-AZ database يوفر database failover.
- RDS Proxy يمكن أن يساعد في connection handling أثناء database failover.

### Solution Thinking

`ALB → ASG Across AZs → Multi-AZ Database`

### Why Not the Alternatives

- **Cross-Region architecture** → أكبر من requirement الخاص بـ regional HA.
- **Single AZ + snapshots** → لا يمنع downtime أثناء AZ failure.
- **Custom S3 + Lambda DR** → unnecessary complexity.

### Key Trap

> **Multi-AZ = Regional High Availability**

> **Cross-Region = Disaster Recovery**

### Quick Recognition

> **Business-critical + minimum downtime → Multi-AZ**

### Final Takeaway

**Choose Multi-AZ because the primary requirement is high availability within the Region.**

---

# Q61 — DynamoDB RPO 15 Minutes

### Problem

DynamoDB data قد يتعرض للـ corruption، والمطلوب recovery إلى نقطة زمنية قريبة.

### Answer

**Enable DynamoDB Point-in-Time Recovery (PITR).**

### Why

- **Primary Requirement:** Recover data to a specific point in time.
- PITR يحتفظ continuous backups.
- يمكن restore table إلى desired point within the supported recovery window.
- مناسب أكثر من snapshots المتباعدة عندما يكون RPO صغيرًا.

### Solution Thinking

`DynamoDB → PITR → Data Corruption → Point-in-Time Restore`

### Why Not the Alternatives

- **Global Tables** → replication/availability وليست point-in-time recovery.
- **Daily S3 backups** → RPO أسوأ من 15 minutes.
- **EBS snapshots** → ليست آلية backup لـ DynamoDB.

### Key Trap

> **Replication/HA ≠ Point-in-Time Recovery**

### Quick Recognition

> **DynamoDB + corruption + specific recovery point → PITR**

### Final Takeaway

**Choose DynamoDB PITR because the requirement is recovery to a recent point in time.**

---

# Q62 — DynamoDB 7-Year Retention

### Problem

Transaction data في DynamoDB يجب الاحتفاظ به لمدة **7 years** بأقل operational overhead.

### Answer

**Use AWS Backup with backup schedules and retention policies.**

### Why

- **Primary Requirement:** Long-term managed backup retention.
- AWS Backup يوفر centralized backup scheduling.
- Retention policies تسمح بإدارة long-term retention.
- لا تحتاج custom Lambda/EventBridge backup architecture.

### Solution Thinking

`DynamoDB → AWS Backup → Scheduled Backups → Long-Term Retention`

### Why Not the Alternatives

- **PITR** → recovery capability وليس الحل المقصود للـ 7-year retention.
- **Manual backup + S3 Lifecycle** → operational overhead أعلى.
- **EventBridge + Lambda + S3** → custom automation غير ضرورية.

### Key Trap

> **PITR = Recovery**

> **AWS Backup = Managed Backup + Retention**

### Quick Recognition

> **Long-Term Backup Retention → AWS Backup**

### Final Takeaway

**Choose AWS Backup because the primary requirement is long-term managed retention.**

---

# Q63 — EC2 Purchasing for Different Environments

### Problem

Production EC2 instances تعمل **24/7**، بينما Dev/Test تعمل فقط أثناء ساعات الاستخدام وسيتم إيقافها عند عدم الحاجة.

### Answer

**Use Reserved Instances for Production and On-Demand Instances for Dev/Test.**

### Why

- **Production:** predictable 24/7 usage → commitment makes sense.
- **Dev/Test:** intermittent usage → flexibility is more valuable.
- Automation التي توقف Dev/Test تقلل ساعات التشغيل، لذلك commitment طويل المدى أقل ملاءمة.

### Solution Thinking

`Production 24/7 → Reserved`

`Dev/Test Intermittent → On-Demand`

### Why Not the Alternatives

- **Spot for Production** → interruption risk غير مناسب لـ production.
- **Reserved for Dev/Test** → resources لا تعمل باستمرار.
- **Spot Blocks** → ليست الاختيار المناسب لهذا usage pattern.

### Key Trap

> **Predictable continuous usage → Reserved**

> **Intermittent usage → On-Demand**

### Quick Recognition

> **Production 24/7 + Dev/Test Scheduled → RI + On-Demand**

### Final Takeaway

**Choose Reserved for continuously running production and On-Demand for intermittent Dev/Test.**

---

# Q64 — Immutable S3 Documents

### Problem

Regulatory requirement تمنع تعديل أو حذف documents بعد تخزينها.

### Answer

**Use S3 Versioning with S3 Object Lock enabled.**

### Why

- **Primary Requirement:** Immutable/WORM storage.
- Object Lock يمنع modification/deletion خلال retention period.
- Versioning يدعم Object Lock ويحافظ على object versions.
- الحل مخصص للـ regulatory immutability requirement.

### Solution Thinking

`Documents → S3 Versioning + Object Lock → WORM`

### Why Not the Alternatives

- **Lifecycle Policy** → archiving/storage management وليس immutability.
- **Versioning + ACL read-only** → access control لا يوفر WORM protection.
- **EFS read-only mount** → لا يضمن immutable storage.

### Key Trap

> **Read-only access ≠ WORM protection**

### Quick Recognition

> **Regulatory + Cannot Modify/Delete → S3 Object Lock**

### Final Takeaway

**Choose S3 Object Lock + Versioning because the primary requirement is immutable document storage.**

---

# Q65 — Protect S3 from Accidental Deletion

### Problem

Audit documents مهمة، والمطلوب حماية البيانات من **accidental deletion**.

### Answer

**Enable S3 Versioning and MFA Delete.**

### Why

- Versioning يحافظ على previous versions عند deletion أو overwrite.
- MFA Delete يضيف MFA requirement لبعض عمليات permanent deletion.
- الحل يستهدف deletion protection مباشرة.

### Solution Thinking

`Important S3 Data → Versioning + MFA Delete → Deletion Protection`

### Why Not the Alternatives

- **MFA on IAM users** → يحسن authentication لكنه لا يوفر S3-specific deletion protection.
- **Lifecycle Policy** → لا تمنع deletion.
- **KMS Encryption** → تحمي confidentiality، وليس deletion.

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

Many Consumers
→ SNS + Multiple SQS Queues

Variable Jobs + Worker Scaling
→ SQS + ASG

On-Prem SMB/NFS + S3
→ S3 File Gateway

Read-Heavy Database
→ Read Replicas

Unpredictable Reads
→ Auto Scaling

ALB HTTP → HTTPS
→ Listener Redirect

DB Credentials + Automatic Rotation
→ Secrets Manager

External CA Certificate
→ Import into ACM

File Upload + Automatic Processing
→ S3 Event → Lambda

Medical Documents + PHI
→ Textract → Comprehend Medical

SQS Duplicate Processing
→ Increase Visibility Timeout

Consistent Low Latency + Cheap Backup
→ Direct Connect + VPN

High Availability Within Region
→ Multi-AZ

DynamoDB Corruption + Specific Recovery Point
→ PITR

Long-Term Backup Retention
→ AWS Backup

EC2 24/7 + Predictable
→ Reserved Instances

EC2 Intermittent
→ On-Demand

Immutable / WORM S3 Data
→ Object Lock + Versioning

S3 Accidental Deletion Protection
→ Versioning + MFA Delete

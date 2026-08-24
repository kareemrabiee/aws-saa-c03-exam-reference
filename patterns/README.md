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
````

---
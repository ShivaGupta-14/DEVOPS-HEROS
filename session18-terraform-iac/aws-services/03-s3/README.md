# S3 - Storage

## What is S3?

S3 (Simple Storage Service) is object storage. We can store any kind of file like images, logs, backups or static websites. It is very durable (11 nines) and we pay for what we store and transfer.

## Buckets

A bucket is the container for files. Bucket names are globally unique across all AWS accounts, but each bucket is created in one region.

```bash
aws s3 mb s3://shiva-devops-demo-bucket --region ap-south-1
```

## Objects

An object is the file plus its metadata. Each object has a key (the full path name, like `logs/2026/app.log`). S3 doesn't have real folders, the `/` in the key just makes it look like folders. Max object size is 5 TB.

```bash
aws s3 cp app.log s3://shiva-devops-demo-bucket/logs/app.log
aws s3 ls s3://shiva-devops-demo-bucket/logs/
```

## Storage classes

| Class | When to use |
|-------|-------------|
| S3 Standard | Frequently accessed data |
| S3 Intelligent-Tiering | Access pattern unknown, AWS moves data for you |
| S3 Standard-IA | Infrequent access but needs fast retrieval |
| S3 One Zone-IA | Infrequent, can be recreated, stored in one AZ |
| S3 Glacier Instant Retrieval | Archive, but needs millisecond access |
| S3 Glacier Flexible Retrieval | Archive, minutes to hours to retrieve |
| S3 Glacier Deep Archive | Cheapest, retrieval in hours |

## Versioning

Versioning keeps every version of an object, so if someone overwrites or deletes a file we can get it back. Once enabled it can only be suspended, not fully turned off. This is also needed for the Terraform state bucket.

```bash
aws s3api put-bucket-versioning --bucket shiva-devops-demo-bucket --versioning-configuration Status=Enabled
```

## Lifecycle policies

Lifecycle rules move or delete objects automatically after some time. For example, move logs to Standard-IA after 30 days, to Glacier after 90 days and delete them after 365 days. They can also clean up old non-current versions so storage cost doesn't keep growing.

## Encryption

Since January 2023 all new objects are encrypted by default with SSE-S3 (keys managed by S3). Other options are SSE-KMS (keys in KMS, more control and audit), DSSE-KMS (two layers) and client-side encryption. Data in transit is protected with HTTPS.

## Bucket policies

A bucket policy is a JSON policy attached to the bucket itself. It is used to allow other accounts, force HTTPS, or make a static website public. Block Public Access is on by default for new buckets, so you have to turn it off on purpose before any public policy works. New buckets also have ACLs disabled by default, so bucket policies are the main way to control access.

## Common use cases

- Storing Terraform remote state (with versioning and encryption)
- Hosting static websites (usually behind CloudFront)
- Backups and log storage
- Storing build artifacts from CI/CD pipelines
- Data lake for analytics

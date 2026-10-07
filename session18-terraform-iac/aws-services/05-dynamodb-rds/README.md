# DynamoDB - NoSQL Database

## NoSQL

DynamoDB is a fully managed NoSQL database. There are no servers to manage and it scales automatically. It is a key-value and document store, so there are no joins and no fixed schema except for the key. It gives single-digit millisecond response times.

## Tables

A table is where data is stored. When creating a table we only define the primary key, everything else is flexible. Capacity mode can be on-demand (pay per request) or provisioned.

```bash
aws dynamodb create-table --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST --region ap-south-1
```

## Items

An item is one record in the table, like a row. Each item can have different attributes. Max item size is 400 KB.

## Attributes

Attributes are the fields of an item, like columns. Types include String (S), Number (N), Boolean, List, Map and more.

```json
{ "UserId": "u101", "OrderDate": "2026-10-01", "Amount": 499, "Status": "SHIPPED" }
```

## Partition key

The partition key (also called hash key) decides which partition the item is stored in. If the table only has a partition key, it must be unique for every item. Pick a key with many different values (like `UserId`) so the load spreads evenly.

## Sort key

The sort key (range key) is optional. With it, the primary key becomes partition key + sort key, so many items can share the same partition key. For example `UserId` + `OrderDate` lets us get all orders of a user sorted by date.

## Use cases

- Terraform state locking (older setups, newer Terraform can lock using S3 itself)
- User sessions and shopping carts
- Gaming leaderboards
- IoT and event data with high traffic

# RDS - Relational Database

## Relational database

RDS (Relational Database Service) is managed SQL databases. AWS handles patching, backups and hardware, we just use the database. Data is stored in tables with fixed schema and we can use joins and transactions.

## Supported engines

- MySQL
- PostgreSQL
- MariaDB
- Oracle
- SQL Server
- Db2
- Amazon Aurora (MySQL and PostgreSQL compatible, AWS's own engine)

## DB instances

A DB instance is the database server running in AWS. We pick an instance class (like `db.t3.micro`), storage type (gp3, io1/io2) and size. We connect using the endpoint AWS gives us, not an IP.

```bash
aws rds describe-db-instances --region ap-south-1 --query "DBInstances[*].Endpoint.Address"
```

## Security

- Put the DB in private subnets (using a DB subnet group)
- Security group allows port 3306/5432 only from the app servers' SG
- Encryption at rest with KMS (must be chosen at creation time)
- SSL/TLS for connections
- Store the password in Secrets Manager instead of hardcoding it

## Backups

Automated backups are enabled by default with a retention of up to 35 days, and they allow point-in-time restore. We can also take manual snapshots, which stay until we delete them.

## Multi-AZ

Multi-AZ keeps a standby copy in another AZ with synchronous replication. If the primary fails, RDS automatically fails over to the standby and the endpoint stays the same. The standby is for high availability, not for reading.

## Read replicas

Read replicas are copies that use asynchronous replication and can serve read traffic. They help scale read-heavy apps and can even be in another region. A replica can be promoted to a standalone DB if needed.

| | Multi-AZ | Read replica |
|---|---|---|
| Purpose | High availability | Read scaling |
| Replication | Synchronous | Asynchronous |
| Can serve reads | No (standard Multi-AZ) | Yes |
| Failover | Automatic | Manual promote |

## Use cases

- Backend database for web apps and APIs
- E-commerce orders and payments where transactions matter
- CMS like WordPress
- Reporting using read replicas

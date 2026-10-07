# IAM - Governance

## What is IAM?

IAM (Identity and Access Management) is the AWS service that controls who can log in and what they are allowed to do. It is global, so it is not tied to any region. IAM itself is free, you only pay for the resources people create.

## Users

A user is one identity, usually a real person or sometimes an app. Each user can have a console password and/or access keys for the CLI.

```bash
aws iam create-user --user-name shiva-dev
```

The root account (the email you signed up with) should not be used for daily work.

## Groups

A group is a collection of users. Instead of attaching policies to every user one by one, we attach them to a group like `developers` or `admins` and add users to it. A user can be in multiple groups, but groups cannot be nested.

## Roles

A role is an identity with permissions but no long-term password or keys. Someone or something "assumes" the role and gets temporary credentials. Common examples:

- An EC2 instance that needs to read from S3
- A Lambda function writing to DynamoDB
- A user from another AWS account getting limited access

## Policies

Policies are JSON documents that say what is allowed or denied. Here is a small one that only allows reading from one bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-demo-bucket",
        "arn:aws:s3:::my-demo-bucket/*"
      ]
    }
  ]
}
```

There are AWS managed policies (like `ReadOnlyAccess`), customer managed policies (ones we write), and inline policies (attached directly to one user/role).

## Permissions

Permissions are the final result of all the policies that apply to an identity. By default everything is denied. An explicit `Allow` gives access, and an explicit `Deny` always wins over any allow.

## Least privilege

Give only the permissions that are actually needed, nothing extra. For example, if a script only uploads files to one bucket, give it `s3:PutObject` on that bucket and not `s3:*` on everything. Start small and add permissions when something fails.

## IAM best practices

- Lock away the root account and enable MFA on it
- Enable MFA for all human users
- Use groups to manage permissions, not individual users
- Use roles for EC2, Lambda etc. instead of storing access keys on servers
- Rotate access keys and delete the ones not in use
- Prefer IAM Identity Center (SSO) for human logins in bigger setups
- Use IAM Access Analyzer to find unused or too-open permissions

## Common use cases

- Giving the dev team access to dev resources only
- Letting an EC2 instance pull files from S3 using an instance role
- CI/CD pipelines (like GitHub Actions) assuming a role with OIDC to deploy with Terraform
- Cross-account access for an auditor or another team

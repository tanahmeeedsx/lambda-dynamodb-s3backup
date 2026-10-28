# AWS Lambda, DynamoDB & S3 Automated Backup

A serverless AWS project that imports user data from JSONPlaceholder into Amazon DynamoDB and exports database records to Amazon S3 using AWS Lambda and EventBridge Scheduler.

## Architecture

```text
JSONPlaceholder API
        |
        v
AWS Lambda
import-jsonplaceholder-users
        |
        v
Amazon DynamoDB
jsonplaceholder-users
        |
        v
AWS Lambda
export-dynamodb-to-s3
        |
        v
Private Amazon S3 Bucket
Timestamped JSON Backups
        ^
        |
Amazon EventBridge Scheduler
Scheduled Backup Trigger
```

Amazon CloudWatch provides logs, invocation metrics, and troubleshooting information for the Lambda functions.

## Tech Stack

- AWS Lambda — serverless execution
- Amazon DynamoDB — NoSQL database
- Amazon S3 — backup object storage
- Amazon EventBridge Scheduler — scheduled execution
- Amazon CloudWatch — logs and monitoring
- AWS IAM — access control and permissions
- Python — Lambda function implementation
- JSONPlaceholder — sample user data API

## Project Structure

```text
lambda-dynamodb-s3backup/
├── README.md
├── .gitignore
├── Screenshots/
├── iam_policies/
│   ├── lambda-backup-export-policy.json
│   └── lambda-import-putitem-policy.json
├── lambda_src/
│   ├── backup_users/
│   │   └── lambda_function.py
│   └── import_users/
│       └── lambda_function.py
└── lambda_zips/
    ├── lambda-backup-to-s3.zip
    └── lambda-import-users.zip
```

## How It Works

### 1. Import Users into DynamoDB

The `import-jsonplaceholder-users` Lambda function retrieves sample user records from JSONPlaceholder and writes them to the `jsonplaceholder-users` DynamoDB table.

The table uses `id` as its numeric partition key.

### 2. Export DynamoDB Records to S3

The `export-dynamodb-to-s3` Lambda function reads records from DynamoDB and exports them as JSON to the configured S3 bucket.

Backup objects are organized under date-based prefixes, following this pattern:

```text
backups/YYYY-MM-DD/<timestamped-backup-file>.json
```

The actual object names depend on the deployed function's implementation.

### 3. Schedule Automated Backups

Amazon EventBridge Scheduler can invoke the export Lambda function on a recurring schedule. The assignment uses a `rate(2 minutes)` expression.

Check the schedule's current state in AWS before expecting automatic backups.

### 4. Monitor Execution

Amazon CloudWatch Logs help troubleshoot function execution. CloudWatch metrics help track invocations, errors, and execution duration.

## IAM & Security

The project includes separate IAM policy documents for importing records and exporting backups.

- Grant each Lambda function only the permissions it requires.
- Keep the S3 backup bucket private.
- Avoid committing AWS access keys, secret keys, or environment files.
- Review resource ARNs and permissions before deployment.
- Confirm that Lambda execution roles have the required permissions for their configured resources.

The policy files are provided for review; verify their resource ARNs and actions against your actual AWS configuration before using them.

## Repository Files

| Path | Purpose |
|---|---|
| `lambda_src/import_users/` | Source code for importing sample users |
| `lambda_src/backup_users/` | Source code for exporting database records |
| `lambda_zips/` | Lambda deployment ZIP archives |
| `iam_policies/` | IAM policy JSON documents |
| `Screenshots/` | AWS console evidence and project walkthrough |

## Screenshots

The `Screenshots/` directory contains 12 screenshots captured during the AWS implementation.

Review these images for the available evidence of configuration, Lambda execution, DynamoDB records, S3 backup objects, scheduling, and monitoring. Each screenshot should be matched to its actual contents before making specific claims about what it proves.

## Key Concepts & Lessons Learned

- Integrating AWS Lambda with DynamoDB and S3.
- Importing API data into a NoSQL database.
- Exporting database records into JSON backup files.
- Using EventBridge Scheduler for recurring serverless tasks.
- Applying IAM permissions to individual workloads.
- Using CloudWatch logs and metrics to troubleshoot executions.
- Understanding the difference between exported data and a complete database recovery configuration.

## Production Considerations

- A two-minute full-table scan is suitable only for a small demonstration, not necessarily for production workloads.
- Consider DynamoDB Point-in-Time Recovery or AWS Backup when recovery requirements justify them.
- Plan for large tables, pagination, execution time limits, retries, and duplicate invocations.
- Configure S3 lifecycle policies and encryption according to retention and security requirements.
- Monitor failed invocations and verify that backups are usable.

## Cleanup

To avoid ongoing AWS charges, review and remove lab resources that are no longer needed:

1. Disable or delete the EventBridge schedule.
2. Delete the Lambda functions when no longer needed.
3. Delete the DynamoDB table if its data is no longer required.
4. Empty the S3 bucket and delete it if appropriate. Handle object versions if versioning is enabled.
5. Review and delete related CloudWatch log groups.
6. Remove lab-only IAM roles and policies only after confirming they are not used elsewhere.

Verify each resource and its dependencies in the AWS console before deleting it.

## Disclaimer

This repository documents a learning project. Resource names, permissions, schedule state, and deployment settings should be verified against the actual AWS environment before reuse.

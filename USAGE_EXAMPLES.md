# Usage Examples

This document provides practical examples of using the IAM policies in this repository for common scenarios.

## Example 1: Setting Up a Data Science Team

### Scenario
You have a team of 5 data scientists who need to:
- Run experiments in SageMaker notebooks
- Train and deploy models
- Access data from specific S3 buckets
- Query data using Athena

### Solution

1. Create an IAM role for the team:

```bash
aws iam create-role \
  --role-name DataScienceTeamRole \
  --assume-role-policy-document file://trust-policy.json
```

Trust policy (`trust-policy.json`):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "sagemaker.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

2. Create and attach the data scientist policy:

```bash
# Create the policy (customize bucket names first)
aws iam create-policy \
  --policy-name DataScienceTeamPolicy \
  --policy-document file://policies/data-scientist/data-scientist.json

# Attach to the role
aws iam attach-role-policy \
  --role-name DataScienceTeamRole \
  --policy-arn arn:aws:iam::123456789012:policy/DataScienceTeamPolicy
```

3. Add CloudWatch monitoring:

```bash
aws iam attach-role-policy \
  --role-name DataScienceTeamRole \
  --policy-arn arn:aws:iam::123456789012:policy/CloudWatchReadOnly
```

## Example 2: ETL Pipeline with Glue

### Scenario
Create an automated ETL pipeline that:
- Runs Glue jobs on a schedule
- Reads from raw data S3 bucket
- Writes to processed data S3 bucket
- Updates Glue Data Catalog

### Solution

1. Create Glue service role:

```bash
aws iam create-role \
  --role-name GlueETLRole \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "glue.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'
```

2. Customize and apply Glue policy:

First, edit `policies/etl-glue/glue-job-execution.json` to replace:
- `your-data-bucket` → `my-raw-data-bucket`
- `your-temp-bucket` → `my-processed-data-bucket`

```bash
aws iam create-policy \
  --policy-name GlueETLPolicy \
  --policy-document file://policies/etl-glue/glue-job-execution.json

aws iam attach-role-policy \
  --role-name GlueETLRole \
  --policy-arn arn:aws:iam::123456789012:policy/GlueETLPolicy
```

3. Create the Glue job:

```bash
aws glue create-job \
  --name daily-etl-job \
  --role GlueETLRole \
  --command '{"Name": "glueetl", "ScriptLocation": "s3://my-scripts/etl.py"}' \
  --default-arguments '{"--TempDir": "s3://my-temp-bucket/temp/"}'
```

## Example 3: CI/CD Pipeline for Microservices

### Scenario
Set up a CI/CD pipeline that:
- Builds Docker containers
- Pushes to ECR
- Deploys to ECS via CodePipeline

### Solution

1. Create CodePipeline service role:

```bash
aws iam create-role \
  --role-name CodePipelineMicroservicesRole \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "codepipeline.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'
```

2. Customize and attach DevOps policy:

Edit `policies/cicd-devops/devops-engineer.json` to update bucket names, then:

```bash
aws iam create-policy \
  --policy-name CodePipelineMicroservicesPolicy \
  --policy-document file://policies/cicd-devops/devops-engineer.json

aws iam attach-role-policy \
  --role-name CodePipelineMicroservicesRole \
  --policy-arn arn:aws:iam::123456789012:policy/CodePipelineMicroservicesPolicy
```

3. Create CodeBuild role for building containers:

```bash
aws iam create-role \
  --role-name CodeBuildContainerRole \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "codebuild.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy \
  --role-name CodeBuildContainerRole \
  --policy-arn arn:aws:iam::123456789012:policy/CodePipelineMicroservicesPolicy
```

## Example 4: Data Analyst Dashboard Access

### Scenario
Business analysts need to:
- View QuickSight dashboards
- Run predefined Athena queries
- Access only production data (read-only)

### Solution

1. Create IAM user group:

```bash
aws iam create-group --group-name BusinessAnalysts
```

2. Apply restricted analyst policy:

```bash
aws iam create-policy \
  --policy-name BusinessAnalystPolicy \
  --policy-document file://policies/data-analyst/business-analyst.json

aws iam attach-group-policy \
  --group-name BusinessAnalysts \
  --policy-arn arn:aws:iam::123456789012:policy/BusinessAnalystPolicy
```

3. Add users to the group:

```bash
aws iam add-user-to-group \
  --user-name alice \
  --group-name BusinessAnalysts

aws iam add-user-to-group \
  --user-name bob \
  --group-name BusinessAnalysts
```

## Example 5: MLOps Pipeline

### Scenario
Set up an end-to-end ML pipeline with:
- Automated model training
- Model registry
- A/B testing deployments
- Monitoring and rollback capabilities

### Solution

1. Create SageMaker pipeline role:

```bash
aws iam create-role \
  --role-name SageMakerMLOpsRole \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "sagemaker.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'
```

2. Attach MLOps specialist policy:

```bash
aws iam create-policy \
  --policy-name MLOpsSpecialistPolicy \
  --policy-document file://policies/ml-lifecycle/mlops-specialist.json

aws iam attach-role-policy \
  --role-name SageMakerMLOpsRole \
  --policy-arn arn:aws:iam::123456789012:policy/MLOpsSpecialistPolicy
```

3. Add KMS encryption for models:

```bash
aws iam attach-role-policy \
  --role-name SageMakerMLOpsRole \
  --policy-arn arn:aws:iam::123456789012:policy/KMSDataEncryption
```

## Example 6: Multi-Account Setup

### Scenario
You have separate AWS accounts for dev, staging, and prod, and need cross-account access.

### Solution

1. In the target account (e.g., production), create a cross-account role:

```bash
# In production account
aws iam create-role \
  --role-name CrossAccountDataScientist \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"AWS": "arn:aws:iam::111111111111:root"},
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-external-id-123"
        }
      }
    }]
  }'

# Attach the policy
aws iam attach-role-policy \
  --role-name CrossAccountDataScientist \
  --policy-arn arn:aws:iam::222222222222:policy/DataScienceTeamPolicy
```

2. In the source account (dev/staging), allow users to assume the role:

```bash
# In dev account
aws iam create-policy \
  --policy-name AssumeProductionRole \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::222222222222:role/CrossAccountDataScientist"
    }]
  }'

aws iam attach-user-policy \
  --user-name data-scientist-user \
  --policy-arn arn:aws:iam::111111111111:policy/AssumeProductionRole
```

3. Use the role:

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/CrossAccountDataScientist \
  --role-session-name my-session \
  --external-id unique-external-id-123
```

## Testing Policies

### Using IAM Policy Simulator

Test if a policy allows specific actions:

```bash
# Test S3 access
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/alice \
  --action-names s3:GetObject s3:PutObject \
  --resource-arns arn:aws:s3:::my-data-bucket/file.csv

# Test SageMaker access
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:role/DataScienceRole \
  --action-names sagemaker:CreateTrainingJob \
  --resource-arns arn:aws:sagemaker:us-east-1:123456789012:training-job/my-job
```

### Dry-Run Testing

Many AWS services support dry-run mode:

```bash
# Test EC2 instance launch
aws ec2 run-instances \
  --dry-run \
  --image-id ami-12345678 \
  --instance-type t2.micro

# Test S3 operations
aws s3 cp file.txt s3://my-bucket/ --dryrun
```

## Common Customizations

### Restricting by IP Address

Add IP-based conditions:

```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": ["192.0.2.0/24", "203.0.113.0/24"]
    }
  }
}
```

### Requiring MFA

Add MFA requirement:

```json
{
  "Condition": {
    "Bool": {
      "aws:MultiFactorAuthPresent": "true"
    }
  }
}
```

### Time-Based Access

Restrict access to business hours:

```json
{
  "Condition": {
    "DateGreaterThan": {"aws:CurrentTime": "2024-01-01T09:00:00Z"},
    "DateLessThan": {"aws:CurrentTime": "2024-01-01T17:00:00Z"}
  }
}
```

## Troubleshooting

### Access Denied Errors

1. Check CloudTrail for the exact action being denied
2. Use IAM Policy Simulator to test the policy
3. Review resource ARNs - ensure they match exactly
4. Check for deny statements that might override allows

### Policy Too Large

If your policy exceeds size limits:
1. Split into multiple managed policies
2. Use wildcards more efficiently
3. Consider using resource tags with conditions
4. Move some permissions to resource-based policies

## Additional Resources

- [AWS Policy Examples](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_examples.html)
- [Testing IAM Policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)
- [Cross-Account Access](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html)

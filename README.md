# AWS Common Policies - Least Privilege

A comprehensive collection of AWS IAM policies following the principle of least privilege for common roles and tasks across different job functions.

## Overview

This repository provides pre-configured, security-focused IAM policies for various AWS use cases, including:
- ETL operations with AWS Glue
- CI/CD pipelines and DevOps workflows
- Machine Learning lifecycle management
- Data Science workflows
- Data Analytics operations

All policies follow the **principle of least privilege**, granting only the minimum permissions required to perform specific tasks.

## Repository Structure

```
policies/
├── etl-glue/          # ETL and AWS Glue policies
├── cicd-devops/       # CI/CD and DevOps policies
├── ml-lifecycle/      # Machine Learning lifecycle policies
├── data-scientist/    # Data Scientist role policies
├── data-analyst/      # Data Analyst role policies
└── common/            # Shared/common policies
```

## Policy Categories

### 1. ETL with AWS Glue (`policies/etl-glue/`)

- **glue-job-execution.json**: Permissions for executing Glue jobs, accessing data catalogs, and S3 operations
- **glue-developer.json**: Full development permissions for creating/managing Glue jobs, crawlers, and databases

### 2. CI/CD and DevOps (`policies/cicd-devops/`)

- **pipeline-operator.json**: Permissions to run and monitor CodePipeline, CodeBuild, and CodeDeploy
- **devops-engineer.json**: Full management of CI/CD infrastructure including pipeline creation and configuration

### 3. ML Lifecycle (`policies/ml-lifecycle/`)

- **ml-engineer.json**: SageMaker permissions for training, deploying, and managing ML models
- **mlops-specialist.json**: Advanced MLOps permissions including experiments, model registry, and pipelines

### 4. Data Scientist (`policies/data-scientist/`)

- **data-scientist.json**: Full data science workflow permissions including notebooks, training, and data access
- **research-scientist.json**: Limited permissions focused on experimentation and analysis

### 5. Data Analyst (`policies/data-analyst/`)

- **data-analyst.json**: Permissions for querying data with Athena, accessing Glue catalogs, and creating visualizations
- **business-analyst.json**: Read-only access for viewing dashboards and querying existing data

### 6. Common Policies (`policies/common/`)

- **s3-read-only.json**: Read-only access to shared S3 buckets
- **cloudwatch-read-only.json**: Read access to CloudWatch logs and metrics
- **kms-data-encryption.json**: KMS permissions for data encryption/decryption

## Usage

### Applying Policies

1. **Identify the appropriate policy** for your use case from the categories above
2. **Customize the policy** by replacing placeholder values:
   - Replace `your-*-bucket` with actual S3 bucket names
   - Update resource ARNs to match your AWS account and regions
   - Adjust specific resource names as needed

3. **Create IAM policy** in AWS Console or using AWS CLI:

```bash
aws iam create-policy \
  --policy-name MyCustomPolicy \
  --policy-document file://policies/data-scientist/data-scientist.json
```

4. **Attach to IAM role or user**:

```bash
aws iam attach-user-policy \
  --user-name john-doe \
  --policy-arn arn:aws:iam::123456789012:policy/MyCustomPolicy
```

### Combining Policies

For comprehensive access, combine role-specific policies with common policies:

**Example: Data Scientist Setup**
```bash
# Attach data scientist policy
aws iam attach-role-policy \
  --role-name DataScientistRole \
  --policy-arn arn:aws:iam::123456789012:policy/DataScientistPolicy

# Attach common CloudWatch read access
aws iam attach-role-policy \
  --role-name DataScientistRole \
  --policy-arn arn:aws:iam::123456789012:policy/CloudWatchReadOnly

# Attach KMS encryption permissions
aws iam attach-role-policy \
  --role-name DataScientistRole \
  --policy-arn arn:aws:iam::123456789012:policy/KMSDataEncryption
```

## Customization Guide

### Before Using These Policies

1. **Review each policy** to ensure it meets your security requirements
2. **Replace placeholder values**:
   - Bucket names: `your-*-bucket`
   - Role ARNs: Update account IDs
   - Resource paths: Adjust to match your naming conventions

3. **Restrict by tags** (optional): Add conditions to limit access based on resource tags:

```json
"Condition": {
  "StringEquals": {
    "aws:ResourceTag/Environment": "production"
  }
}
```

4. **Limit by region** (optional): Restrict actions to specific AWS regions:

```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": ["us-east-1", "us-west-2"]
  }
}
```

### Example Customizations

#### Restrict S3 Access to Specific Prefix
```json
{
  "Resource": [
    "arn:aws:s3:::my-data-bucket/team-a/*"
  ]
}
```

#### Add Time-Based Access
```json
{
  "Condition": {
    "DateGreaterThan": {"aws:CurrentTime": "2024-01-01T00:00:00Z"},
    "DateLessThan": {"aws:CurrentTime": "2024-12-31T23:59:59Z"}
  }
}
```

## Best Practices

1. **Start with minimum permissions**: Begin with the most restrictive policy and add permissions as needed
2. **Use IAM roles over users**: Prefer roles with temporary credentials over long-term user credentials
3. **Enable MFA**: Require multi-factor authentication for sensitive operations
4. **Regular audits**: Review and update policies regularly using AWS Access Analyzer
5. **Tag resources**: Use consistent tagging for better policy management
6. **Test policies**: Use IAM policy simulator before deploying to production
7. **Monitor usage**: Enable CloudTrail and review access patterns

## Policy Validation

Before deploying policies, validate them using:

### AWS CLI Policy Validator
```bash
aws accessanalyzer validate-policy \
  --policy-document file://policies/data-scientist/data-scientist.json \
  --policy-type IDENTITY_POLICY
```

### IAM Policy Simulator
```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:role/MyRole \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::my-bucket/my-key
```

## Security Considerations

- **Least Privilege**: All policies grant minimum permissions required
- **No wildcards in resources**: Where possible, specific resources are referenced
- **Condition keys**: Policies use conditions to further restrict access
- **Service-specific**: Permissions are scoped to specific services
- **Pass Role restrictions**: IAM PassRole actions are limited to specific service roles

## Contributing

To add new policies or improve existing ones:

1. Follow the existing policy structure
2. Ensure policies follow least privilege principles
3. Test policies thoroughly before submitting
4. Document the use case and permissions granted
5. Update this README with new policy information

## License

This repository is provided as-is for educational and reference purposes. Review and customize all policies according to your organization's security requirements before use.

## Additional Resources

- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS Policy Generator](https://awspolicygen.s3.amazonaws.com/policygen.html)
- [IAM Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html)
- [Least Privilege Principle](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)
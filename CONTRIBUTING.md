# Contributing Guidelines

Thank you for your interest in contributing to the AWS Common Policies repository! This document provides guidelines for adding new policies or improving existing ones.

## How to Contribute

### Reporting Issues

If you find a security issue or incorrect permission in a policy:

1. **For security vulnerabilities**: Please report privately (do not create a public issue)
2. **For policy improvements**: Open an issue describing:
   - Which policy is affected
   - What the problem is
   - Suggested improvement
   - Use case or scenario

### Adding New Policies

1. **Fork the repository** and create a new branch
2. **Add your policy** in the appropriate directory
3. **Follow the structure** outlined below
4. **Test the policy** thoroughly
5. **Update documentation**
6. **Submit a pull request**

## Policy Structure Guidelines

### File Organization

Place policies in the appropriate directory:
- `policies/etl-glue/` - AWS Glue and ETL operations
- `policies/cicd-devops/` - CI/CD and DevOps tools
- `policies/ml-lifecycle/` - Machine Learning workflows
- `policies/data-scientist/` - Data science operations
- `policies/data-analyst/` - Data analysis and BI
- `policies/common/` - Shared/cross-functional policies

### Naming Conventions

- Use lowercase with hyphens: `my-policy-name.json`
- Be descriptive: `sagemaker-notebook-user.json` not `policy1.json`
- Include the role/function: `glue-developer.json`, `ml-engineer.json`

### Policy Format

All policies must follow this structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DescriptiveStatementId",
      "Effect": "Allow",
      "Action": [
        "service:Action1",
        "service:Action2"
      ],
      "Resource": [
        "arn:aws:service:region:account:resource-type/resource-name"
      ]
    }
  ]
}
```

### Required Elements

1. **Statement ID (Sid)**: Use descriptive names
   - ✅ Good: `"Sid": "S3DataBucketReadAccess"`
   - ❌ Bad: `"Sid": "Statement1"`

2. **Actions**: List specific actions, avoid wildcards where possible
   - ✅ Good: `["s3:GetObject", "s3:PutObject"]`
   - ⚠️ Use sparingly: `["s3:*"]`

3. **Resources**: Be as specific as possible
   - ✅ Good: `"arn:aws:s3:::my-specific-bucket/*"`
   - ❌ Bad: `"*"`

4. **Conditions**: Use conditions to further restrict access
   ```json
   "Condition": {
     "StringEquals": {
       "aws:RequestedRegion": ["us-east-1", "us-west-2"]
     }
   }
   ```

## Least Privilege Principles

### 1. Grant Minimum Permissions

Only include permissions that are absolutely necessary:

```json
// ✅ Good - specific actions
"Action": [
  "s3:GetObject",
  "s3:ListBucket"
]

// ❌ Bad - overly permissive
"Action": "s3:*"
```

### 2. Specific Resources

Avoid `"Resource": "*"` except when required by the service:

```json
// ✅ Good - specific resources
"Resource": [
  "arn:aws:s3:::my-data-bucket/*",
  "arn:aws:s3:::my-data-bucket"
]

// ⚠️ Only when necessary
"Resource": "*"  // Some actions like ecr:GetAuthorizationToken require this
```

### 3. Use Conditions

Add conditions to further restrict access:

```json
{
  "Effect": "Allow",
  "Action": "iam:PassRole",
  "Resource": "arn:aws:iam::*:role/service-role/AmazonSageMaker*",
  "Condition": {
    "StringLike": {
      "iam:PassedToService": "sagemaker.amazonaws.com"
    }
  }
}
```

### 4. Separate Read and Write Permissions

Create separate policies for read and write operations when appropriate:

```json
// read-only-policy.json
{
  "Action": ["s3:GetObject", "s3:ListBucket"]
}

// write-policy.json
{
  "Action": ["s3:PutObject", "s3:DeleteObject"]
}
```

## Testing Requirements

Before submitting a policy, ensure:

### 1. JSON Validation

```bash
# Validate JSON syntax
cat policy.json | python -m json.tool
```

### 2. AWS Policy Validation

```bash
# Use AWS Access Analyzer
aws accessanalyzer validate-policy \
  --policy-document file://policy.json \
  --policy-type IDENTITY_POLICY
```

### 3. Functional Testing

Create a test role and verify the policy works:

```bash
# Create test role
aws iam create-role --role-name TestRole --assume-role-policy-document file://trust-policy.json

# Attach your policy
aws iam put-role-policy --role-name TestRole --policy-name TestPolicy --policy-document file://your-policy.json

# Test with IAM Policy Simulator
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:role/TestRole \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::test-bucket/test-key

# Clean up
aws iam delete-role-policy --role-name TestRole --policy-name TestPolicy
aws iam delete-role --role-name TestRole
```

### 4. Security Review

Check for common security issues:
- No overly permissive wildcards
- No unnecessary `"Resource": "*"`
- Proper use of conditions
- No hardcoded account IDs (use placeholders)
- No sensitive data in policy

## Documentation Requirements

When adding a new policy, you must:

### 1. Update README.md

Add your policy to the appropriate section:

```markdown
### X. Category Name (`policies/category/`)

- **your-policy.json**: Description of what permissions are granted and use case
```

### 2. Add Policy Header Comment (Optional)

Consider adding metadata in a separate `.md` file:

```markdown
# Policy Name

**File**: `your-policy.json`
**Category**: Data Science
**Use Case**: Allows data scientists to run experiments

## Permissions Granted
- SageMaker: Create and run training jobs
- S3: Read/write to specific data buckets
- CloudWatch: Write logs

## Example Usage
[Link to example]
```

### 3. Provide Usage Example

Add an example to `USAGE_EXAMPLES.md`:

```markdown
## Example X: Your Use Case

### Scenario
Describe the scenario...

### Solution
Provide step-by-step instructions...
```

## Code Review Process

All contributions go through review:

1. **Automated checks**: JSON validation, syntax checking
2. **Security review**: Verify least privilege principles
3. **Functional review**: Ensure policy serves stated purpose
4. **Documentation review**: Check for complete documentation

## Style Guide

### JSON Formatting

- Use 2 spaces for indentation
- Keep arrays on multiple lines if more than one element
- Alphabetize actions within a statement
- Order statements logically (read before write, core before auxiliary)

Example:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadAccess",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::bucket-name",
        "arn:aws:s3:::bucket-name/*"
      ]
    },
    {
      "Sid": "WriteAccess",
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::bucket-name/*"
      ]
    }
  ]
}
```

### Placeholder Values

Use consistent placeholders:
- S3 buckets: `your-data-bucket`, `your-logs-bucket`
- Account ID: `123456789012` or `*` if account-agnostic
- Regions: Use `*` unless region-specific
- Role names: `your-role-name` or `YourRoleName`

## Common Mistakes to Avoid

1. **Too permissive**: Using `"*"` unnecessarily
2. **Missing conditions**: Not restricting PassRole actions
3. **Wrong resource ARN format**: Check AWS documentation
4. **Mixing service actions**: Keep related actions together
5. **Not testing**: Always test policies before submitting

## Getting Help

- Open an issue for questions
- Review existing policies as examples
- Check AWS IAM documentation
- Use AWS Policy Simulator for testing

## License and Legal

By contributing, you agree that your contributions will be licensed under the same terms as the project.

## Recognition

Contributors will be acknowledged in the project. Thank you for helping make AWS IAM policies more accessible and secure!

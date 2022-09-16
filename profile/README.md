# Usage of AWS CodePipeline as CI/CD

## Introduction

Products App is a simple app created as the final assignment for the final exam of the course IoT Cloud Systems (University of Catania). The code and all the needed files are gathered in repositories inside the organization called "iot-cloud-system-final-assignment-mc".

The goal of this assignment was to use AWS CodePipeline to support the development and deployment of a cloud infrastructure on AWS, using AWS SAM and AWS CloudFormation to manage every AWS resource.

The resources that are deployed in this infrastucture are the following:
- AWS S3 for AWS SAM templates (needed to store the artifacts created by AWS SAM)
- 3 AWS CloudFormation Stacks containing the following resources:
  - AWS Cognito resources (Userpool, app client, admin groups, admin user)
  - AWS CloudFront to host the website
  - AWS S3 to store the static HTML files of the website
  - AWS Lambda Layer with all the common libraries/dependencies for the lambdas deployed in this architecture
  - AWS DynamoDB tables for Products and Orders
- 3 AWS CloudFormation Stacks for Auth, Products and Orders Lambdas, each containing:
  - 1 AWS Lambda Function definition
  - 1 AWS IAM Role and 1 AWS IAM Policy
- 1 AWS CloudFormation Stack for the API Gateway containing:
  - 1 AWS API Gateway definition
  - 1 AWS Lambda permission for each endpoint defined in the API Gateway definition
- 5 AWS CloudFormation Stacks for the AWS CodePipelines resources containing the following resources:
 - 1 AWS CodePipeline definition
 - 1 AWS CodeBuild definition
 - Required AWS IAM Roles and AWS IAM Policies
 - 1 AWS S3 to store the artifacts


It follows the list of the repositories in this organization with a brief description of their role:
- bootstrap-configuration contains the AWS SAM template to deploy the "static" resources (DynamoDB tables, Cognito Userpools etc.). This is the only section of the infrastructure that is not combined with a CodePipeline.
- web-client contains the code of the frontend and the AWS SAM template to create the AWS CodePipeline used to store the static html pages of the frontend in the S3 served by AWS CloudFront
- api-gateway contains the AWS SAM template to deploy the API Gateway resource and the AWS SAM template to create the AWS CodePipeline used to update it
- auth-lambda contains the code and the AWS SAM template to deploy the Authorizer lambda (needed to authorize the access to the api gateway to logged users) and the AWS SAM template to create the AWS CodePipeline used to update it
- lambda-products contains the code and the AWS SAM template to deploy a bunch of Lambdas used to interact with the Products DynamoDB table and the AWS SAM template to create the AWS CodePipeline used to update it
- lambda-orders contains the code and the AWS SAM template to deploy a bunch of Lambdas used to interact with the Orders DynamoDB table and the AWS SAM template to create the AWS CodePipeline used to update it

## Deploy the infrastructure

### Prerequisites

- AWS CLI
- AWS SAM
- GitHub OAUTH Token

### How to deploy

#### Script

For a simple and fast deployment just download and run the following script (BASH/SH required):

```
https://raw.githubusercontent.com/iot-cloud-system-final-assignment-mc/bootstrap-configuration/main/deploy_everything.sh

ADMIN_EMAIL=xxx@xxx.xx
GITHUB_OAUTH_TOKEN=XXXX
AWS_REGION=XXX
SAM_S3_BUCKET=XXX

bash deploy_everything.sh $ADMIN_EMAIL $GITHUB_OAUTH_TOKEN $AWS_REGION $SAM_S3_BUCKET

```
NOTE: Change all the Xs with the real information.
NOTE2: If you want to automatically create the SAM_S3_BUCKET, leave it empty.



#### Steps-by-step


## Destroy the infrastructure

### Script

```
https://raw.githubusercontent.com/iot-cloud-system-final-assignment-mc/bootstrap-configuration/main/destroy_everything.sh

AWS_REGION=xxx-xxxx-xx

bash destroy_everything.sh $AWS_REGION

```
NOTE: Change all the Xs with the real information.



#### Steps-by-step

1) Setup the AWS REGION local env:

```
AWS_REGION=xxx-xxxx-xx
```

2) Empty the S3 bucket inside the bootstrap stack:

```
BOOTSTRAP_S3_BUCKET=$(aws cloudformation list-exports --query "Exports[?Name=='Cloud-Systems-IoT-ApplicationSiteBucket'].Value" --output text)
aws s3 rm s3://${BOOTSTRAP_S3_BUCKET} --recursive --region $1
```

3) Delete every stack with the resources of the infrastructure:

```
aws cloudformation delete-stack --stack-name bootstrap-resources --region $1
aws cloudformation delete-stack --stack-name lambda-products --region $1
aws cloudformation delete-stack --stack-name lambda-orders --region $1
aws cloudformation delete-stack --stack-name auth-lambda --region $1
aws cloudformation delete-stack --stack-name api-gateway --region $1
```

4) Empty the S3 bucket in every CodePipeline stack:

```
PIPELINE_BUCKET=$(aws cloudformation list-stack-resources --stack-name web-client --query "StackResourceSummaries[?LogicalResourceId == 'PipelineBucket'].PhysicalResourceId" --output text)
aws s3 rm s3://${PIPELINE_BUCKET} --recursive --region $1

PIPELINE_BUCKET=$(aws cloudformation list-stack-resources --stack-name lambda-products-pipeline --query "StackResourceSummaries[?LogicalResourceId == 'PipelineBucket'].PhysicalResourceId" --output text)
aws s3 rm s3://${PIPELINE_BUCKET} --recursive --region $1

PIPELINE_BUCKET=$(aws cloudformation list-stack-resources --stack-name lambda-orders-pipeline --query "StackResourceSummaries[?LogicalResourceId == 'PipelineBucket'].PhysicalResourceId" --output text)
aws s3 rm s3://${PIPELINE_BUCKET} --recursive --region $1

PIPELINE_BUCKET=$(aws cloudformation list-stack-resources --stack-name auth-lambda-pipeline --query "StackResourceSummaries[?LogicalResourceId == 'PipelineBucket'].PhysicalResourceId" --output text)
aws s3 rm s3://${PIPELINE_BUCKET} --recursive --region $1

PIPELINE_BUCKET=$(aws cloudformation list-stack-resources --stack-name api-gateway-pipeline --query "StackResourceSummaries[?LogicalResourceId == 'PipelineBucket'].PhysicalResourceId" --output text)
aws s3 rm s3://${PIPELINE_BUCKET} --recursive --region $1
```

5) Delete every stack of CodePipelines resources:

```
aws cloudformation delete-stack --stack-name web-client --region $1
aws cloudformation delete-stack --stack-name lambda-products-pipeline --region $1
aws cloudformation delete-stack --stack-name lambda-orders-pipeline --region $1
aws cloudformation delete-stack --stack-name auth-lambda-pipeline --region $1
aws cloudformation delete-stack --stack-name api-gateway-pipeline --region $1
```

6) Optional: Empty and delete the S3 used to store AWS SAM artifacts

```
SAM_S3_BUCKET=XXX
aws s3 rm s3://${SAM_S3_BUCKET} --recursive --region $1
aws s3 delete-bucket --bucket ${SAM_S3_BUCKET}
```

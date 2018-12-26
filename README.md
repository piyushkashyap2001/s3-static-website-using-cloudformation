# s3-static-website-using-cloudformation
## Overview
This repository constains an AWS Cloudformation template and other necessary lambda functions to automate infrastructure deployment to host a static website. This cloudformation stack also creates a CI/CD pipeline for automated deployment of static website and it also setups a Elasticsearch service for S3 access logs analysis and visualization .
## Architecture
![Preview](https://raw.githubusercontent.com/piyushkashyap2001/s3-static-website-using-cloudformation/master/architecture.png)
## Prerequisites

- Static website (html/css/javascript). (I prefer [Hexo](https://hexo.io/) static site generator.)
- A registered domain name for website.
- Should have [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) installed & configured and git [setup](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up.html) for AWS CodeCommit on the local machine.
- Good knowledge of Amazon web services.

## AWS services

- Route 53 - DNS service and domain registration for website
- Cloudfront - CDN, HTTPS
- S3 buckets - Website content, www redirect, access logs, CodePipeline artifacts, lambda zip files
- Lambda functions - S3 logs to cloudwatch, cloudwatch logs stream to elastisearch, certificate, delete elasticsearch indices
- Cloudwatch - Collects s3 access logs, trigger events
- Elasticsearch/Kibana - Logs analysis and visualization
- Cognito - Authentication for Kibana
- Certificate Manager - Free certificates
- IAM - Security, roles, permissions
- Cloudformation - Infrastructure Management
- Codecommit - Git repository
- Codepipeline - Orchestrates git and code build
- Codebuild - Code dependency installation

## How to run

**Step 1** - As the lamdba function in the main template uses a S3 bucket to store the code, Create a S3 bucket first using this [template](https://github.com) either from AWS console or by below AWS CLI command.

```bash
aws cloudformation create-stack --stack-name {stackname} --template-body file://{path_to_template_file}
```

**Step 2** - Upload all the lambda zip code files to the bucket created in above step using following AWS CLI command.

```bash
 aws s3 cp {folder path}/ s3://{bucketname}/lambdas/ --recursive
```

**Step 3** - After that main stack can be created using this [template](https://github.com). It might take around 40 to 50 minutes to create all the resources.

```bash
aws cloudformation create-stack --stack-name {stackname} --template-body file://{path_to_template_file} --capabilities CAPABILITY_IAM --parameters ParameterKey=DomainName, ParameterValue={basedomain} ParameterKey=PreExistingHostedZoneDomain, ParameterValue={hosted zone}  ParameterKey=PreExistingHostedZoneId, ParameterValue={hosted zone id} ParameterKey=ProjectName, ParameterValue={project name}
```

Use following commands to track the stack events and output.

```bash
aws cloudformation describe-stacks --stack-name {stackname}
aws cloudformation describe-stack-events --stack-name {stackname}
```

**Step 4** - Once the stack is created, the output of stack will provide the site bucket name, Kibana url, CodeCommit url, Cognito user pool id.

- Use AWS Codecommit url to clone the repository.
- Replace the bucket name in buildspec.yml file with the site bucket name.
- Create a new user profile in the user pool to access Kibana using following command.

```bash
aws cognito-idp admin-create-user --user-pool-id {userpoolid} --username {username} --user-attributes Name=email_verified,Value=true,Name=email,Value={emailid}  --region {region} --temporary-password {temp password}
```

- Finally login into Kibana using the credentials created in the above step and configure the Kibana to use the Elasticsearch indices.

## Important Notes

- As soon as the code is pushed to AWS Codecommit, AWS Codepipeline is triggered by the push , then it uses AWS CodeBuild to build the code & finally AWS CodeBuild copies the file to S3 site bucket. The static website code must have a buildspec.yml file at the root level of folder structure in order to have AWS CodeBuild install the dependencies and move the files to S3 site bucket.

- The template fully automates the provisioning of certificate including validation. It uses DNS validation method to validate the certificate. In order to use ACM Certificate with Cloudfront distribution, the certificate will be created in us-east-1 region only.

- The template has been tested in us-east-1(North Virginia) and ap-south-1(Mumbai) regions. It should work in other regions as well.

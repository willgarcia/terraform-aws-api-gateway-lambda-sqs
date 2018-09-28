# Terraform AWS API Gateway Lambda SQS

### Terraform module for AWS API Gateway Lambda SQS infrastructure
[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](https://opensource.org/licenses/MIT)
![stability-stable](https://img.shields.io/badge/stability-stable-brightgreen.svg)
![Commitizen-friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)
## Table of Contents
* [Features](#features)
* [Usage](#usage)
* [Deployment](#deployment)
* [Example](#example)

## Features
Terraform module which deploys a serverless HTTP endpoint backed by AWS API Gateway, Lambda & SQS
 
**API Gateway**

This module is created with a single stage that is given as parameter.
The path is `/api/messages`.
I will expand this in later versions to be user provided.

**Lambda**

This module is created with full customization by user.
Exports S3 bucket to allow usage by multiple Lambda's
- This module by default, if created allows accompanying Lambda access to SQS if SQS entry is provided as parameters.


**SQS**

This module is optional. Lambda is created with W permission for SQS to allow Lambda to add/read/delete messages from queue.
- This module by default, if created has no permissions added.

## Usage
```hcl-terraform
module "api-gateway-lambda-sqs" {
  source  = "crisboarna/api-gateway-lambda-sqs/aws"
  version = "1.0.0"

  # insert the required variables here
}
```

## Deployment
1. Run build process to generate Lambda ZIP file locally to match `lambda_zip_path` variable path
2. Provide all needed variables from `variables.tf` file or copy paste and change example below
3. Create/Select Terraform workspace before deployment
4. Run `terraform plan -var-file="<.tfvars file>` to check for any errors and see what will be built
5. Run `terraform apply -var-file="<.tfvars file>` to deploy infrastructure

**Example Deployment Script**
```js
#!/usr/bin/env bash

if [[ ! -d .terraform ]]; then
  terraform init
fi
if ! terraform workspace list 2>&1 | grep -qi "$ENVIRONMENT"; then
  terraform workspace new "$ENVIRONMENT"
fi
terraform workspace select "$ENVIRONMENT"
terraform get
terraform plan -var-file=$1
terraform apply -var-file=$1
```

## Example
```hcl-terraform
module "api_lambda_sqs" {
  source  = "crisboarna/terraform-aws-api-gateway-lambda-sqs"
  version = "v1.0.0"

  #Global
  region = "eu-west-1"
  project = "Awesome Project"
   
  #API Gateway
  api_gw_method = "POST"

  #Lambda
  lambda_function_name = "Awesome Endpoint"
  lambda_description = "Awesome HTTP Endpoint Lambda"
  lambda_runtime = "nodejs8.10"
  lambda_handler = "dist/bin/lambda.handler"
  lambda_timeout = 30
  lambda_code_s3_bucket = "awesome-project-bucket"
  lambda_code_s3_key = "awesome-project.zip"
  lambda_code_s3_storage_class = "ONEZONE_IA"
  lambda_code_s3_bucket_visibility = "private"
  lambda_zip_path = "../../awesome-project.zip"
  lambda_memory_size = 256
  
  #SQS
  sqs_queue_names = ["SQS_QUEUE_NAME"]
  sqs_queue_delay_seconds = [0]
  sqs_queue_max_message_sizes = [2046]
  sqs_queue_message_retention_seconds = [34565]
  sqs_queue_receive_wait_time_seconds = [10]
  sqs_queue_fifos = [true]
  sqs_queue_content_based_deduplications = [true]
  sqs_dead_letter_max_receive_counts = [3]
  
  #Tags
  tags = {
    project = "Awesome Project"
    managedby = "Terraform"
  }
  
  #Lambda Environment variables
  environmentVariables = {
    NODE_ENV = "production"
  }
}
```
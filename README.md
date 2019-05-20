# Certified AWS Associate Developer Notes

### My notes in preparation for the 2019 AWS developer associate exam 

## Table of contents

- AWS Fundamentals
    - [IAM: Identity Access & Management](aws-fundamentals/iam.md)
    - [EC2: Virtual Machines](aws-fundamentals/ec2.md)
    - [ELB: Elastic Load Balancers](aws-fundamentals/elb.md)
    - [ASG: Auto Scaling Group](aws-fundamentals/asg.md)
    - [EBS Volumes](aws-fundamentals/ebs.md)
    - RDS: Relational Database Service
    - Route 53
    - ElastiCache
    - VPC: Virtual Private Cloud
    - S3 Buckets

- AWS Deep Dive
    - CLI: Command Line Interface
    - SDK: Software Development Kit
    - IAM Roles & Policies
    - Elastic Beanstalk
    - CICD: CodeCommit, CodePipeline, CodeBuild, CodeDeploy
    - CloudFormation
    - CloudWatch
    - SQS
    - SNS
    - Kinesis
    - Serverless: Lambda
    - Serverless: DynamoDB
    - API Gateway
    - Cognito
    - ECS: Elastic Container Service
    - ECR: Elastic Container Registry
    - Fargate

- [Exam Preperation](#exam-preparation)


## Exam Preparation

- Exam details
    - Two question types:
        - Multiple Choice
        - Multiple response
    - Minimum passing score: 720/1000
    - Domains:
        - Deployment: CICD, Beanstalk, Serverless
        - Security: each service deep-dive + dedicated section
        - Development with AWS Services: Serverless, API, SDK, & CLI
        - Refactoring: Understand all the AWS services for the best migration
        - Monitoring and Troubleshooting: CloudWAtch, CloudTrail, X-Ray

    - Exam Guide:
        - https://aws.amazon.com//certification/certified-developer-certificate/

- EC2 + IAM Exam Checklist
  * Know how to SSH into EC2 (and change .pem file permissions) 
  * Know how to properly use security groups 
  * Know the fundamental differences between private vs public vs elastic IP 
  * Know how to use User Data to customize your instance at boot time 
  * Know that you can build custom AMI to enhance your OS 
  * EC2 instances are billed by the second and can be easily created and thrown away, welcome to the cloud!  
  Maybe on Exam:
  * Availability zones are in geographically isolated data centers
  * IAM users are NOT defined on a per-region basis
  * If you are getting a permission error exception when trying to SSH into your linux instance, then the key is missing chmod 400 permissions
  * If you are getting a network timeout when trying to SSH into your EC2 instance, then your security groups are misconfigured
  * Security groups reference IP address, CIDR block, Security group, but NOT DNS name

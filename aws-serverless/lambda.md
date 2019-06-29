# AWS Lambda

Why use AWS Lambda?
- Avoid limitations of EC2 Servers:
    - Virtual Servers in the Cloud
    - Limited by RAM and CPU
    - Continuously running
    - Scaling means intervention to add / remove servers
- Lambda provides:
    - Virtual functions
    - no servers to manage
    - Quick, short executions
    - Run on-demand
    - Scaling is fully automated

### Benefits of AWS Lambda
- Easy Pricing:
- Pay per request and compute time
- Free tier of 1,000,000 AWS Lambda requests and 400,000 GBs of compute time
- Integrations with the entire AWS Stack
- Integrated with many programming languages
- Easy monitoring through AWS CloudWatch
- Easy to get more resources per functions (up to 3GB of RAM)
- Increasing RAM will also improve CPU and network

AWS Lambda language support:
- Node.js (JavaScript)
- Python
- Java (Java 8 compatible) • C# (.NET Core)
- Golang
- C# / Powershell

### AWS Lambda pricing
- You can find overall pricing information here: https://aws.amazon.com/lambda/pricing/
- Pay per calls:
- First 1,000,000 requests are free
- $0.20 per 1 million requests thereafter ($0.0000002 per request)
- Pay per duration: (in increment of 100ms)
- 400,000 GB-seconds of compute time per month if FREE
- == 400,000 seconds if function is 1GB RAM
- == 3,200,000 seconds if function is 128 MB RAM
- After that $1.00 for 600,000 GB-seconds
- It is usually very cheap to run AWS Lambda which makes it very popular

### AWS Lambda configuration
- Timeout: default 3 seconds, max of 300s (Note: new limit 15 minutes) - Environment variables
- Allocated memory (128M to 3G)
- Ability to deploy within a VPC + assign security groups
- IAM execution role must be attached to the Lambda function
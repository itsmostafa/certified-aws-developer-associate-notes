# SDK: Software Development Kit

If you want to perform actions on AWS directly from your application's code without using a CLI, you can use an SDK

Official SDKs:
- Java
- .NET
- Node.js
- PHP
- Python
- Ruby
- C++

#### SDK Takeaways
- AWS SDK are required when coding against AWS Services such as DynamoDB
- Fact: AWS CLI uses the Python SDK (boto3)
- The exam expects you to know when you should use an SDK
- If you don’t specify or configure a default region, then us-east-1 will be chosen by default

#### SDK Credentials Security

- It’s recommend to use the default credential provider chain
- The default credential provider chain works seamlessly with:
  - AWS credentials at ~/.aws/credentials (only on our computers or on premise)
  - Instance Profile Credentials using IAM Roles (for EC2 machines, etc...)
- Environment variables (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY)
- Overall, NEVER EVER STORE AWS CREDENTIALS IN YOUR CODE.
- Use IAM Roles if working from within AWS Services to inherit credentials

#### Exponential Backoff

- Any API that fails because of too many calls needs to be retried with Exponential Backoff
- These apply to rate limited API
- Retry mechanism is included in SDK API calls
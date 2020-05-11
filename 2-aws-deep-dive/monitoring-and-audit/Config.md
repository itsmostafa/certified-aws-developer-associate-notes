# AWS Config

- AWS Config is usefull to do compliance checks. 
- It is possible to run compliance checks periodicaly. 
- AWS Config requires an IAM Role 
   - Read-Only permission to AWS Resources
   - Write access to S3 Bucket 
   - Publish access to SNS
- It is possible to use CloudTrail with Config for deeper insight into resources. 
- CloudTrail can also check access to Config as it is auditing purpose. 

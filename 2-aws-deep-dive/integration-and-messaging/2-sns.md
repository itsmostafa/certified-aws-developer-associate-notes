# SNS: Simple Notification Service

SNS is a fully managed messaging service for both application to application (A2A) and application to person (A2P) communication.

- The event producer only sends a message to one SNS topic at a time
- Many event receivers can be created to listen to an SNS topic notification
- Limits:
    - 10,000,000 subscriptions per topic
    - 100,000 topics
- Subscriber types:
    - SQS
    - HTTP / HTTPS (with delivery retries)
    - Lambda
    - Emails
    - Mobile Notifications
    - SMS text messages

## SNS integrates with most AWS Services
- AWS Services can send data directly to SNS for generating notifications:
    - CloudWatch (for alarms)
    - Auto Scaling Groups
    - S3 buckets (for events)
    - CloudFormation (for state changes)

## How to publish a Topic
- Simple:
    - Create a topic
    - Create a subscription
    - Public to the topic

- Publish directly from an application
    - Create an endpoint in your application
    - Publish to the platform endpoint
    - This works with:
        - Google GCM
        - Apple APNS
        - Amazon ADM

## Security
- Encryption:
    - In-flight encryption using HTTPS API
    - At-rest encryption using KMS keys
    - Client-side encryption if the client wants to perform encryption/decryption itself

- Access Controls: IAM policies to regulate access to the SNS API

- SNS Access Policies (similar to S3 Bucket policies)
    - Useful for mult account access to SNS topics
    - Useful for allowing other services to write to an SNS topic


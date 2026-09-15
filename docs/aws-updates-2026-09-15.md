# AWS documentation updates - 2026-09-15

Total updates: 22

## bedrock

- [Amazon Bedrock Managed Knowledge Base now supports multimodal embeddings for video, audio, and image content with TwelveLabs Marengo 3.0](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-bedrock-managed-knowledge-base-multimodal-embeddings-twelvelabs-marengo/) — published 2026-09-11T15:13:00+00:00
  
  <p>AWS announces the availability of TwelveLabs Marengo 3.0 as an embedding model in Amazon Bedrock Managed Knowledge Base, enabling customers to create multimodal embeddings for video, audio, and image content. Amazon Bedrock Managed Knowledge Base already supports media search by transcribing audio and video to text and generating text-based embeddings—Marengo 3.0 goes further by encoding visual scenes, speech, and video cues directly into multimodal embeddings, capturing meaning that transcri

- [OpenAI GPT-6 Astra is now generally available on Amazon Bedrock](https://aws.amazon.com/about-aws/whats-new/2026/09/openai-gpt-6-astra-on-amazon-bedrock/) — published 2026-09-08T22:49:00+00:00
  
  <p>Today, AWS announces the general availability of GPT-6 Astra from OpenAI on Amazon Bedrock. The latest and most capable model from OpenAI to date, GPT-6 Astra brings deeper reasoning and judgment, professional-quality writing and design, and advanced computer and browser use to demanding business workflows. It supports a context window of up to 1 million input tokens and can produce output aligned with organizational voice, templates, and standards. The Amazon Bedrock inference engine deliver

- [AWS announces Nx Plugin for AWS for scaffolding full-stack applications](https://aws.amazon.com/about-aws/whats-new/2026/09/nx-plugin-for-aws/) — published 2026-09-08T08:00:00+00:00
  
  <p>Version 1.0 of the Nx Plugin for AWS, an open source toolkit for scaffolding full-stack applications on AWS, is now available. AI assistants can stand up an application on AWS in minutes, but rarely get security, observability, and type-safety right in one pass. The plugin extends Nx, an open source, language-agnostic build system for monorepos, with generators that each build one part of an application on request, alongside the infrastructure to run it.</p>  <p>Generators cover AI agents and

- [Web Search on Amazon Bedrock is now available in AWS GovCloud (US-West)](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-bedrock-web-aws-govcloud/) — published 2026-09-02T17:52:00+00:00
  
  <p>The Web Search built-in server-side tool on Amazon Bedrock is now available in AWS GovCloud (US-West), helping bring grounded web results to compliance-sensitive government and public-sector workloads. Web Search helps supported OpenAI GPT models ground responses with information from the web. Responses include citations to the sources the model used so users can trace each claim back to its web origin. This can be especially valuable whenever an answer depends on information that changes ove

- [AWS Config now supports 60 new resource types](https://aws.amazon.com/about-aws/whats-new/2026/09/aws-config-new-resource-types/) — published 2026-09-02T15:00:00+00:00
  
  <p>AWS Config now supports 60 additional AWS resource types across key services including Amazon Bedrock,&nbsp; Amazon EC2, Amazon SageMaker, and AWS Organizations. This expansion provides greater coverage over your AWS environment, enabling you to more effectively discover, assess, audit, and remediate an even broader range of resources.</p>  <p>With this launch, if you have enabled recording for all resource types, then AWS Config will automatically track these new additions. The newly support


## cloudformation

- [AWS Lambda now supports direct read configuration for Amazon S3 Files](https://aws.amazon.com/about-aws/whats-new/2026/09/aws-lambda-direct-read-s3files/) — published 2026-09-11T17:00:00+00:00
  
  <p>AWS Lambda now supports <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-files-performance.html#s3-files-performance-how">direct read</a>&nbsp;configuration for Amazon S3 Files, letting you configure which storage your functions read from: S3 Files high-performance storage or your S3 bucket. With this launch, you can optimize the throughput and latency of file reads for your Lambda functions based on your application requirements.</p>  <p>Customers use S3 Files with Lambda fu

- [Amazon API Gateway now supports 1 MB execution logs with configurable delivery destinations](https://aws.amazon.com/about-aws/whats-new/2026/09/amazon-api-gateway-1-mb-execution-logs/) — published 2026-09-10T20:00:00+00:00
  
  <p>Amazon API Gateway now supports configurable delivery destinations and larger log events for REST API execution logs. Previously, execution logs were delivered to a single API Gateway-managed CloudWatch Logs log group with log events truncated at 1 KB, limiting visibility into request and response data.<br /> <br /> You can now route execution logs up to 1 MB to your own Amazon CloudWatch Logs log groups, Amazon S3 buckets, or Amazon Data Firehose streams, and deliver to multiple destinations

- [Amazon CloudFront announces API support for flat-rate pricing plans](https://aws.amazon.com/about-aws/whats-new/2026/09/cloudfront-flat-rate-pricing-plans-api/) — published 2026-09-03T18:00:00+00:00
  
  <p>Starting today, customers can subscribe and manage flat-rate pricing plans programmatically using the AWS CLI, AWS SDKs, CloudFormation, CDK, or the PricingPlanManager API.</p>  <p>CloudFront flat-rate plans give you one monthly price covering global content delivery, WAF, DDoS, DNS, logging, and edge compute, with no usage-based overage charges regardless of traffic spikes or attacks. Previously, customers could only subscribe to flat-rate pricing plans using the console, which required manu

- [AWS Lambda now supports SnapStart for container image functions](https://aws.amazon.com/about-aws/whats-new/2026/07/aws-lambda-snapstart-container/) — published 2026-09-02T09:00:00+00:00
  
  <p>Starting today, AWS Lambda supports SnapStart for functions packaged as <a href="https://docs.aws.amazon.com/lambda/latest/dg/images-create.html" rel="noopener noreferrer" target="_blank">container images</a>, reducing startup times from several seconds to as low as sub-second. Lambda SnapStart is an opt-in capability that makes it easier for you to build highly responsive and scalable applications without provisioning resources or implementing complex performance optimizations.</p>  <p>Custo


## cloudwatch

- [Internet Monitor publishes local health events to Amazon EventBridge](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-IM-EventBridge-integration.html) — published 2026-05-20T19:00:00+00:00
  
  Internet Monitor now publishes local health events to Amazon EventBridge, in addition to overall (global) health events.           Local health events have an impact type of <code class="code">LOCAL_AVAILABILITY</code> or <code class="code">LOCAL_PERFORMANCE</code>.


## ec2

- [Permissions checks for EC2 Fast Launch](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/win-fast-launch-configure.html#win-fast-launch-permissions) — published 2026-08-14T19:00:00+00:00
  
  Added guidance on how Amazon EC2 validates your permissions when you enable 					EC2 Fast Launch with a launch template, including a security consideration for 					using the <code class="code">$Latest</code> or <code class="code">$Default</code> launch template 					version.


## eks

- [EFA support for static capacity in EKS Auto Mode](https://docs.aws.amazon.com/eks/latest/userguide/create-node-class.html#static-network-interfaces) — published 2026-07-21T19:00:00+00:00
  
  Added support for configuring EFA network interfaces on EKS Auto Mode NodeClass resources. You can now statically define network interfaces at instance launch for high-performance distributed training and inference workloads.

- [AWS managed policy updates](https://docs.aws.amazon.com/eks/latest/userguide/security-iam-awsmanpol.html) — published 2026-05-12T19:00:00+00:00
  
  Added new API action <code class="code">ec2:DescribeInstanceTypes</code> to <code class="code">AmazonEBSCSIDriverPolicy</code> to enable the EBS CSI Driver to dynamically retrieve volume limit/card information at runtime.

- [AWS managed policy updates](https://docs.aws.amazon.com/eks/latest/userguide/security-iam-awsmanpol.html) — published 2026-02-02T19:00:00+00:00
  
  Removed the "eks" prefix requirement in the name of the target instance profile for the <code class="code">iam:GetInstanceProfile</code> permission in <code class="code">AmazonEKSServiceRolePolicy</code>. This allows Amazon EKS Auto Mode to validate and utilize custom instance profiles in NodeClasses without requiring the "eks" naming prefix.


## lambda

- [AWS managed policy update](https://docs.aws.amazon.com/lambda/latest/dg/security-iam-awsmanpol.html#lambda-security-iam-awsmanpol-updates) — published 2026-06-19T19:00:00+00:00
  
  Lambda added a new AWS managed policy (<code class="code">AWSLambdaNetworkConnectorOperatorPolicy</code>) to grant permissions to create and administer elastic network interface (ENI) resources managed by the Lambda Network Connector. For more information, see <a href="https://docs.aws.amazon.com/lambda/latest/dg/security-iam-awsmanpol.html#lambda-security-iam-awsmanpol-updates">Lambda updates to AWS managed policies</a>.

- [AWS managed policy update](https://docs.aws.amazon.com/lambda/latest/dg/security-iam-awsmanpol.html#lambda-security-iam-awsmanpol-updates) — published 2026-06-10T19:00:00+00:00
  
  Lambda updated the <code class="code">AWSLambdaManagedEC2ResourceOperator</code> managed policy to add Amazon CloudWatch permissions (<code class="code">logs:CreateLogGroup</code>, <code class="code">logs:CreateLogStream</code>, <code class="code">logs:PutLogEvents</code>). For details, see <a href="https://docs.aws.amazon.com/lambda/latest/dg/security-iam-awsmanpol.html#lambda-security-iam-awsmanpol-updates">Lambda updates to AWS managed policies</a>.

- [Self-managed S3 code storage](https://docs.aws.amazon.com/lambda/latest/dg/configuration-self-managed-storage.html) — published 2026-04-30T19:00:00+00:00
  
  Lambda now supports self-managed S3 buckets for function and layer code storage. You can configure Lambda to reference your code directly from your S3 bucket, eliminating Lambda-managed storage limits. For details, see <a href="https://docs.aws.amazon.com/lambda/latest/dg/configuration-self-managed-storage.html">Self-managed S3 code storage</a>.

- [AWS managed policy update](https://docs.aws.amazon.com/lambda/latest/dg/security-iam-awsmanpol.html#lambda-security-iam-awsmanpol-updates) — published 2026-04-23T19:00:00+00:00
  
  Lambda updated the <code class="code">AWSLambdaManagedEC2ResourceOperator</code> managed policy to add the <code class="code">ec2:DescribeVpcEncryptionControls</code> permission. For details, see <a href="https://docs.aws.amazon.com/lambda/latest/dg/security-iam-awsmanpol.html#lambda-security-iam-awsmanpol-updates">Lambda updates to AWS managed policies</a>.


## rds

- [RDS Custom for Oracle](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/RDS-Custom-for-Oracle-end-of-support.html) — published 2026-03-31T19:00:00+00:00
  
  End of support notice: On March 31, 2027, AWS will end support for Amazon RDS Custom for Oracle. For more information, see <a href="https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/RDS-Custom-for-Oracle-end-of-support.html">RDS Custom for Oracle end of       support</a>.


## s3

- [Amazon S3 Express One Zone adds new AWS managed policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WhatsNew.html) — published 2026-04-03T19:00:00+00:00
  
  Amazon S3 Express One Zone added new AWS-managed policies called  				<code class="code"> AmazonS3ExpressFullAccess</code> and <code class="code">AmazonS3ExpressReadOnlyAccess</code>. For more information, see  				<a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-express-one-zone-security-iam-awsmanpol.html">AWS managed policies for Amazon S3 Express One Zone</a>.


## secrets-manager

- [Update to AWS managed policy](https://docs.aws.amazon.com/secretsmanager/latest/userguide/document-history.html) — published 2026-06-02T19:00:00+00:00
  
  The <code class="code">AWSSecretsManagerClientReadOnlyAccess</code> managed policy now includes <code class="code">BatchGetSecretValue</code> and <code class="code">ListSecrets</code> permissions. For information, see <a href="https://docs.aws.amazon.com/secretsmanager/latest/userguide/reference_available-policies.html#security-iam-awsmanpol-updates">Secrets Manager updates to AWS managed policies</a>.


## step-functions

- [8 new AWS SDK service integrations](https://docs.aws.amazon.com/step-functions/latest/dg/awssdk-release-history.html) — published 2026-06-03T19:00:00+00:00
  
  Added AWS DevOps Agent Service, AWS Interconnect, AWS Marketplace Discovery, AWS Security Agent, AWS Sustainability, AWS User Experience Customization, Amazon S3 Files, Amazon SimpleDB v2.


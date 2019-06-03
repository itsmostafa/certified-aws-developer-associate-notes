# CodePipeline

- Continuous delivery
- Visual workflow
- Source: GitHub, CodeCommit, Amazon S3
- Build: CodeBuild, Jenkins, etc...
- Load Testing: 3rd par ty tools
- Deploy: AWS CodeDeploy, Beanstalk, CloudFormation, ECS...
- Made of stages:
    - Each stage can have sequential actions and / or parallel actions
    - Stages examples: Build, Test, Deploy, Load Test, etc...
    - Manual approval can be defined at any stage

#### Codepipline Artifacts
- Each pipeline stage can create ”artifacts”
- **Artifacts** are passed stored in Amazon S3 and passed on to the next stage

#### Troublshooting
- CodePipeline state changes happen in **AWS CloudWatch Events**, which can in return create SNS notifications.
- Ex: you can create events for failed pipelines • Ex: you can create events for cancelled stages
- If CodePipeline fails a stage, your pipeline stops and you can get information in the console
- AWS CloudTrail can be used to audit AWS API calls
- If Pipeline can’t perform an action, make sure the “IAM Service Role”
attached does have enough permissions (IAM Policy)
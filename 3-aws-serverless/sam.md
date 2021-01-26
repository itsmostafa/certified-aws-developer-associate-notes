# AWS SAM

- SAM = Serverless Application Model
- It is a framework for developing and deploying serverless applications
- All configuration is done in YAML code. From this CloudFormation code is generated
- Supports anything than CloudFormation supports
- SAM can use CodeDeploy
- SAM can help run Lambda, API Gateway and DynamoDB locally
- Transform header indicates it's a SAM template:
    - `Transform: 'AWS::Serverless-2016-10-31'`
- Serverless resource types:
    -`AWS::Serverless::Function`
    -`AWS::Serverless::Api`
    -`AWS::Serverless::SimpleTable`
- Package and deploy:
    - `aws cloudformation package`
    - `sam package`
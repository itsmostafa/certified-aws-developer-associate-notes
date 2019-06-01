# CICD: Continuous Integration and Deployment

- Why use CICD?
    - Ideally, you'd want to set up a CICD to help you automate multiple steps to automate builds, push code to a repository and then deploy to your updated code to AWS.
    - This is a faster, efficient way that also helps minimize potential mistakes as opposed to running multiple manual steps
    - Automate deployement to different stages (dev, staging, and production)
    - May add manual approvals when needed
- To be a proper AWS developer, you'd need to learn CICD

- CICD Services in AWS
    - AWS CodeCommit: storing our code (similar to Github)
    - AWS CodePipeline: automatig our pipeline from code to ElasticBeanstalk
    - AWS CodeBuild: To build and test code
    - AWS CodeDeploy: deploying code to EC2 fleets (not Beanstalk)

- Continuous Integration
    - Developers push code to a repository (Github, Bitbucket, CodeCommit, etc)
    - A testing or build server checks the code as soon as it's pushed to the repository (CodeBuild, Jenkins CI, etc)
    - The developer gets feedback about the tests and checks that have passed or failed
    - Benefits of CI:
        - Find bugs and fix it early on
        - Deliver fast as soon as the code is tested
        - Deploy frequently
        - Unblocked developers are happy

- Continuous Delivery
    - Ensure that the software can be released reliably whenever needed
    - Ensures deployment happen often and quick
    - Ability to shift away from "one release every 3 months" to "5 releases a day"
    - Automated deployments

Orchestration == CICD

#### CodeCommit
- **Version Control** is the ability to understand changes that happened to the code over time (and possibly roll back)
- This is enabled by using a version control system such as Git
- A git repository can live on your machine or on a centrol online repository
- Benefits:
    - Collaborate with a team of developers
    - Make sure the code is backed-up somewhere
    - Makes sure it's fully viewable and auditable
- AWS CodeCommit
    - private git repositories
    - No size limits on the repositories (scale seamlessly)
    - Fully managed and highly available
    - The code is only in your AWS cloud account
    - Increased security and compliance
    - Secure (encrypted access control, etc)
- CodeCommit Security
    - Interactions are done using git
    - Authentication with Git
        - SSH Keys: AWS Users can configure SSH keys in their IAM Console
        - HTTPS: Done through AWS CLI Authentication helper or generate HTTPS credentials
        - MFA: Multi Factor Authentication
    - Authorization with Git
        - IAM Policies manage user / roles rights to the repositories
    - Encryption
        - Repositories are automatically encrypted at rest using KMS
        - Encrypted in transit (can only user HTTPS or SSH  both secure)
    - Cross Account access:
        - Do not share your SSH keys
        - Do not share your AWS credentials
        - Use IAM Role in your AWS account and use AWS STS (with AssumeRole API)  
- Github vs. CodeCommit
    - Similarities
        - Github and CodeCommit can be integrated with AWS CodeBuild
        - Both support HTTPS and SSH authentication
    - Differences
        - Security
            - Github: Github Users
            - CodeCommit: AWS IAM users & roles
        - Hosted
            - Github: hosted by Github
            - CodeCommit: managed & hosted by AWS
        - UI
            - Github UI is fully featured
            - CodeCommit is minimal
- CodeCommit notifications
    - You can trigger notifications in CodeCommit using AWS SNS(Simple Notification Service) or AWS Lambda or AWS CloudWatch Event Rules
    - Use cases for notifications SNS / AWS Lambda notifications:
        - Deletion of branches
        - Trigger for pushes that happens in master branch
        - Notify external Build System
        - Trigger AWS Lambda function to perform codebase analysis (maybe credentials got committed in the code?)
    - Use cases for CloudWatch Event Rules:
        - Trigger for pull request updates (created / updated / deleted / commented)
        - Commit comment events
        - CloudWatch Event Rules goes into an SNS topic

**NOTE**: As of May 31, 2019, Use the "Old Experience" UI for now as the new UI has bugs!

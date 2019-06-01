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

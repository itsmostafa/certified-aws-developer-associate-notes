# CloudFormation

- Currently, we have been doing a lot of manual work
- All this manual work will be very tough to reproduce:
    - In another region
    - In another AWS account
    - Within the same region if everything was deleted
- Wouldn’t it be great, if all our infrastructure was... code?
- That code would be deployed and create / update / delete our
infrastructure

#### What is CloudFormation?
- CloudFormation is a declarative way of outlining your AWS Infrastructure, for any resources (most of them are supported).
- For example, within a CloudFormation template, you want to:
    - I want a security group
    - I want two EC2 machines using this security group
    - I want two Elastic IPs for these EC2 machines
    - I want an S3 bucket
    - I want a load balancer (ELB) in front of these machines
    - Then CloudFormation creates those for you, in the right order, with the exact configuration that you specify

**Note**: This is an introduction to CloudFormation
- It can take over 3 hours to properly learn and master CloudFormation
- This section is meant so you get a good idea of how it works
- We’ll be slightly less hands-on than in other sections
- We’ll learn everything we need to answer questions for the exam
- The exam does not require you to actually write CloudFormation
- The exam expects you to understand how to read CloudFormation

#### Benefits of CloudFormation
- Infrastructure as code
    - No resources are manually created, which is excellent for control
    - The code can be version controlled for example using git
    - Changes to the infrastructure are reviewed through code
- Cost
    - Each resources within the stack is stagged with an identifier so you can easily see how much a stack costs you
    - You can estimate the costs of your resources using the CloudFormation template
    - Savings strategy: In Dev, you could automation deletion of templates at 5 PM and recreated at 8 AM, safely
- Productivity
    - Ability to destroy and re-create an infrastructure on the cloud on the fly
    - Automated generation of Diagram for your templates!
    - Declarative programming (no need to figure out ordering and orchestration)
- Separation of concern: create many stacks for many apps, and many layers. Ex:
    - PC stacks
    - Network stacks
    - App stacks
- Don’t re-invent the wheel
    - Leverage existing templates on the web!
    - Leverage the documentation

#### How CloudFormation works
- Templates have to be uploaded in S3 and then referenced in CloudFormation
- To update a template, we can’t edit previous ones. We have to re- upload a new version of the template to AWS
- Stacks are identified by a name
- Deleting a stack deletes every single artifact that was created by
CloudFormation.

#### Deploying CloudFormation templates
- Manual way:
    - Editing templates in the CloudFormation Designer
    - Using the console to input parameters, etc
- Automated way:
    - Editing templates in a YAML file
    - Using the AWS CLI (Command Line Interface) to deploy the templates
    - Recommended way when you fully want to automate your flow

#### CloudFormation Building Blocks
- Templates components (one course section for each):
    1. Resources: your AWS resources declared in the template (MANDATORY)
    2. Parameters: the dynamic inputs for your template
    3. Mappings: the static variables for your template
    4. Outputs: References to what has been created
    5. Conditionals: List of conditions to perform resource creation
    6. Metadata
- Templates helpers:
    1. References
    2. Functions
 
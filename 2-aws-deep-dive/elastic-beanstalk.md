# Elastic Beanstalk

#### *Elastic Beanstalk* is a developer centric view of deploying application on AWS.
- A managed service
  - Instance configuration
  - OS is handled by Beanstalk
  - Deployment strategy is configurableut performed by Beanstalk
  - Application code configurable
- It will leverage all the AWS components that we have gone over thus far:
  - EC2
  - ASG
  - ELB
  - RDS
  - Etc..
- Elastic Beanstalk is free but you pay for the underlying instances
- Three architecture models:
  - Single instance deployment: good for developers
  - LB + ASG: great for production or staging web applications
  - ASG only: great for non-web apps in production
- Elastic Beanstalk has three components:
  - Application
  - Application Version (Each deployment gets assigned a version)
  - Environment name (dev, staging, prod): free naming
- You deploy application versions to environments and can promote application versions to the next environment
- Rollback feature to previous application versions
- Full control over the lifecycle of environments
- Support for many platforms:
  - Go
  - Java
  - Python
  - Node.js
  - Ruby
  - Single Container Docker
  - Multi Container Docker
  - Preconfigure Docker
  - Write your own custom platforms (If the any of the above is not supported)

#### Blue / Green Deployment
- This is not a direct feature of Elastic Beanstalk
- Zero downtime and release facility
- Create a new staging environment and deploy your newest version there
- The new environment (green) can be validated independently and roll back if there's issues
- Route 53 can be setup using weighted policies to redirect a little bit of traffic to the staging environment
- Using the elastic beanstalk console, you can "swap URLs" when with the testing environment
- This is a manual feature, it's not directly embedded in EB

#### Elastic Beanstalk Extensions
- A zip file containing our code must be deployed to Elastic Beanstalk
- All the parameters set in the UI can be configured with code using files
- Requirements:
  - in the .ebextensions/ directory in the root of source code
  - YAML / JSON format
  - .config extensions (example: logging.config)
  - Able to modify some default settings using: option_settings
  - Ability to add resources such as RDS, ElastiCache, DynamoDB, etc...
- Resources managed by .ebextensions get deleted if the environment goes away
- The .ebextensions folder goes to the root of your project

#### Elastic Beanstalk CLI
- We can install an additional CLI called the “EB cli” which makes working with Beanstalk from the CLI easier
- Basic commands are:
  - eb create
  - eb status
  - eb health
  - eb events
  - eb logs
  - eb open
  - eb deploy
  - eb config
  - eb terminate
- It’s helpful for your automated deployment pipelines!
# AWS Step Functions

- Step Functions allow to build visual workflows which are used to orchestrate Lambda Functions
- Workflow is represented as a JSON state machine
- Features:
    - Sequence
    - Parallel execution
    - Conditions
    - Timeouts
    - Error handling
- Can also integrate with EC2, ECS, on premise servers, API Gateway
- Maximum execution time is 1 year
- Possibility to implement human approval feature
- Use cases:
    - Order fulfillment
    - Data processing
    - Web applications
    - Any workflow
- When designing a Step Function we get an aspect of visualization (flow diagram)
- The execution can be visually represented on this diagram
- Any state can encounter errors:
    - State machine definition issues (example. no matching rules in choice state)
    - Task failures (example: an exception in a Lambda function)
    - Transient failures (example: network partition events)
- By default: when a state reports an error, the execution of the Step Functions fails entirely
- Failures can be retried:
    - Exponential backoff: *IntervalSeconds*, *MaxAttempts*, *BackoffRate*
    - Move on - Catch: *ErrorEquals*, *Next*
- Best practice: include data in the error message

## Standard vs Express Step Functions

- Standard Function
    - Maximum duration: 1 year
    - Exactly-one workflow execution
    - Execution start rate: 2000 per second
    - Price per state transition: more expensive in general
- Express Function 
    - Maximum duration: 5 minutes
    - At-least-once workflow execution
    - Execution start rate: 100_000 per second
    - Price per number of execution: much cheaper in general
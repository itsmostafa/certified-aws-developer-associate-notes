# SQS: Simple Queue Service

SQS is a service that manages and operates message oriented middleware. It enables you to decouple and scale microservices, distributed systems, and serverless applications.

What's a queue?
    - A form of asynchronouse service to service communication used in multiple application architectures.
    - Messages are stored on teh queue until they are processed and deleted

## Standard Queue
    - Oldest offering from AWS (over 10 years old)
    - Fully managed service
    - Scales from 1 message per second to 10,000 per second
    - Default retention of messages: 4 days
        - A maximum of 14 days
    - There is no limit to how many messages can be in the queue
    - Low latency (10 ms on publish and recieve)
    - Horizontal scaling in terms of number of consumers
    - Can have duplicate messages
    - Can have messages out of order (best effort ordering)
    - Limitation of 256kb per message sent


## Delay Queue
    - Delay a message so a consumer doesn't see it immediately
        - up to 15 minute delay
    - Default is 0 seconds making messages available right away
    - You can set a default at queue level
    - YOu can override the default using the **DelaySeconds** parameter

### Producing Messages
    - Define the body
    - Add message attributes (metadata - optional)
    - Optionally can provide Delay Delivery
    - You recieve back:
        - Message identifier
        - MD5 hash of the body

### Consuming Messages
    - Poll SQS for messages
        - receive up to 10 messages at a time
    - Next step is to process the message within the visibility timeout
    - Finally, delete the message using the message ID & receipt handle

### Visibility timeout
    - When a consumer polls a message from a queue, the message is *invisible* to the other consumers for a defined period called **Visibility Timeout**
        - Can be set between 0 seconds to 12 hours
        - Default 30 seconds
        - If it takes 15 minutes or greater and consumer fails to process the message, you must wait before processing the message again
        - IF set to 30 seconds or lower and consuemr needs time to process the message, another consumer will receive the message and will be processed more than once
    - **ChangeMessageVisibilty**
        - An API to change the visibility while processing a message
    - **DeleteMessage**
        - An API to tell SQS the message was successfully processed

### Dead Letter Queue
    - To be continued
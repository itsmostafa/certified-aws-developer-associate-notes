# Integration and Messaging

When we start deploying multiple applications, they will inevitably need to communicate with one another
- There are two patterns of application communication
    - Synchronous communication
        - Direct application to application
        - Synchronous between applications can be problematic if there are sudden spikes of traffic
    - Asynchronous / Event based communication
        - application to queue to application
- **Decouple** your applications:
    - SQS: queue model
    - SNS: pub/sub model
    - Kinesis: real-time streaming model
- These services can scale independently from your application

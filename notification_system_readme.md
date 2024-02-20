## Assumptions
1. We are interested in sending as well as receiving millions of notifications per minute in an end-to-end system.
2. Message Size: We assume that the size of each notification message is within reasonable limits for efficient processing and network transmission.
3. Notification Channels: We assume that notifications can be sent through various channels such as email, SMS, push notifications, etc., and each channel may have its own requirements and constraints.
4. Authentication and Authorization: We assume that the system will integrate with authentication and authorization services to ensure that only authorized users can send notifications and access sensitive data.
5. Reliability: We assume a certain level of reliability and delivery guarantees for notifications, with mechanisms in place to handle failures and retries.
6. Scalability: We assume that the system needs to scale horizontally to handle increasing loads and accommodate future growth.
7. Monitoring and Analytics: We assume the need for comprehensive monitoring and analytics capabilities to track system performance, identify bottlenecks, and optimize resource utilization.
8. Compliance: We assume the need to comply with relevant regulations and standards governing data privacy, security, and communication protocols.
9. Cost Considerations: We assume the need to optimize resource usage and minimize costs while ensuring high performance and reliability.
10. Integration Points: We assume the need to integrate with existing systems and services for user management, content personalization, and reporting.
11. Traffic Patterns: We assume that the traffic pattern for notifications is dynamic and can vary significantly over time, with occasional spikes in demand.

## Functional Requirements
1. Notification Handling: The system must be able to send and receive notifications.
2. High Throughput: The system should handle at least 1 million notifications per minute.

    ```1 Million Notifications per minute = 50 to 60 microseconds per notification```

3. Asynchronous Processing: Notifications should be processed asynchronously to avoid blocking the sender.
4. Message Queue Integration: Utilize Apache Kafka Cluster for message queuing and streaming.
5. Load Balancing: Employ load balancing to distribute incoming requests evenly across multiple instances.
6. Scalability: The system should be horizontally scalable to handle varying loads.
7. Fault Tolerance: Ensure the system remains operational even in the face of failures or errors.
8. Monitoring: Implement comprehensive monitoring to track system health and performance metrics.
9. Alerting: Set up alerts for anomalies, high error rates, or queue backlogs.
10. Security: The system should enforce authentication and authorization to ensure that only authorized users can send notifications and access sensitive data.
11. Cloud Agnostic Design: Avoid vendor lock-in by designing the system to be compatible with multiple cloud providers.

## Non-Functional Requirements
1. Performance: Notifications should be sent with minimal delay and high throughput.
2. Reliability: Ensure that notifications are delivered reliably without loss.
3. Scalability: The system should scale horizontally to accommodate increasing loads.
4. Availability: The system should be highly available, with minimal downtime.
5. Resilience: Design the system to withstand failures and recover gracefully.
6. Cost-effectiveness: Optimize resource usage to minimize operational costs.
7. Maintainability: The system should be modular and well-documented, with clear separation of concerns to facilitate maintenance and updates.
8. Compliance: Ensure compliance with relevant regulations and standards.
9. Usability: Provide an intuitive interface for managing notifications and monitoring system health.


## Microservices Architecture

### Notification Service:
* Responsible for receiving incoming notification requests from users.
* Enqueues notifications into the message queuing system for asynchronous processing.
* Allow TTL message, Sharding, Region replication and Read-Write Quorums.
* Validates and sanitizes notification content.
* Applies rate limiting and throttling policies.
* Use Async/Non-Blocking operations to maintain connection whilst keeping the thread count low and by extension, reducing resource usage (CPU, Memory, I/O)
* Suggest using ```Deno```(TypeScript on Rust), similar to Node.JS or ```netty```(Java) or ```Tokio```(Rust)

### Message Queuing Service:
* Utilizes a scalable message queuing system. (Kafka Cluster)
* Supports message partitioning for parallel processing (Topics).
* Allows for different messages to be served with different priority. 
(For example, Update vs OTP Notification)
* Stores and manages notification messages for asynchronous processing.
* Ensures reliable delivery, fault tolerance and buffers bursty traffic.
* Suggest use of ```Apache Kafka``` due to extremely high scalability.

### Worker Services/Stream processing engine:
* Consumes messages from the message queue and processes notifications.
* Identifies which notification is intended for which client.
* Can be divided into multiple types based on notification channels or message types.
* Scales horizontally to handle varying loads depending on the number of messages held in queue.
* Implements retry and error handling mechanisms.
* Recommend use of ```Kafka Streams``` (Used by Line Messaging App and Trivago) or some alternative written in Rust like ```Arroyo``` to ensure high throughput.

### Load Balancer:
* Distributes incoming traffic across multiple instances of the notification service and worker services.
* Monitors the health and availability of backend services.
* Provides high availability and fault tolerance.
* We may use either ```NGINX```, ```HAProxy``` or if we're building the system using GCP, Azure, AWS, etc. then their respective services.

### Authentication and Authorization Service:
* Handles user authentication and authorization for sending notifications.
* Integrates with Identity and Access Management (IAM) solutions.
* We may choose to use ```KeyCloak``` with JWT or any other IAM solution.

### Monitoring and Logging Service:
* Collects and aggregates system metrics, logs, and events.
* May provide real-time monitoring and alerting for system health and performance.
* Integrates with monitoring tools like Prometheus, Grafana, or ELK stack.

### Caching Service:
* Stores frequently accessed data to improve performance and reduce latency.
* Improve read speed when accessing client registries.
* Utilizes in-memory caching or distributed caching solutions like Redis or Memcached.

### API Gateway:
* Provides a unified interface for external clients to interact with the notification system.
* Handles authentication, request routing, and rate limiting.
* Acts as a proxy to backend services and enforces security policies.

### Database Service:
* Stores persistent data such as notification history.
* Utilizes a scalable and fault-tolerant database system like PostgreSQL, MongoDB, or Amazon DynamoDB.
* Supports data replication and backup for data durability.

### Configuration Service:
* Manages configuration parameters and settings for the notification system.
* Supports dynamic configuration updates without service restarts.
* Centralizes configuration management for easier maintenance.

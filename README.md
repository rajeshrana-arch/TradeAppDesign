# Trading Platform
## Rajesh Kumar Rana

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

In trade brokerage system, brokerage application receives huge number of requests from different users on multiple channels. These requests are always come from multiple platforms such as mobile, website or over a call etc. Please refer the architecture diagram for reference:
![alt text](https://github.com/rajeshrana-arch/TradeAppDesign/blob/master/Trade%20Design%20System.drawio.svg)

- Type some Markdown on the left
- See HTML in the right
- ✨Magic ✨

## Components
##### Order Validation Service
This service is responsible for validating the request before entering into our trading systems. The service would be responsible for performing the below checks:
- Ensure that the user's account exists and is active.
- Check if the stock symbol provided in the request is valid and exists in the market. 
- Ensure that the stock symbol format is correct and matches the expected format
- Validate that the transaction type (buy/sell) is provided and is one of the expected values
- Check if the transaction request is made during market hours.Restrict trading activities outside market hours if required.
- Implement limits on the maximum size of the order that can be placed to prevent large orders from impacting market stability
- Provide clear and informative error messages if any validation checks fail.
- Confirm the successful execution of the transaction and provide relevant details to the user.

After successfully validating the request, generate a valid **TradeID and store it in the TradeRequest Table**. Replicate TradeRequest for OrderStatus to ensure clear separation of data insertion and retrieval, maintaining integrity and consistency in the stock market system.

##### Risk Assemement Service (RMS)
Service Responsible for identify the validation action based upon the type of OrderType(Buy/Sell). It would be responsible mainly for:
 - In case of Buy, check wallet balance in Broker system and block amount in wallet.
 - In case of Sell, check for number of ShareQuantity in Holding Account and block number of share quantity in Holdings.
 - In case of Buy, block amount in wallet.
 - In case of any failure revert back to user with approporate answer.

Based upon the stock name and action Type(Buy/Sell) it would be forwarding the respective queue in our kafka system for processing.

#### Matching Engine
Our component triggers fetching all buy/sell orders. For buy orders, it sorts by highest to lowest value; for sell orders, lowest to highest. It matches buy vs. sell orders, splitting if needed. Matched orders are processed; remaining ones are pushed back to the queue.Based on market cap, it routes to Large, Mid, Small, or Microcap queues so that we can do parallel processing. Later on these messages from queue is processed by Message Processor Engine.

#### Executor Engine Service
An integration layer connects our trading system with the Stock Exchange (Black Box System). After submitting a request, the Order's status is updated in the orderStatus queue. A message processor forwards the status to Analytics Service for system analysis and Notification Service for updating the OrderRequest table with appropriate status (Success, Failure, or Rejection).

#### Notification Service:
This lightweight service is tasked with notifying users about holidays, maintenance downtime, or unexpected system behavior. It efficiently delivers messages to keep users informed and ensure smooth operation of the system.
#### Design Key Points:
### High availability
- We'll adopt a **containerized approach** to build our applications.
- Utilizing **Kubernetes as our orchestrator**, we implement a strategy of maintaining an **idle replication set to three for each application**, ensuring enhanced reliability and fault tolerance in our system architecture.
- Using **Rancher** we will simplifies Kubernetes cluster setup and management across various cloud providers and on-premises setups.
-  **Kafka brokers**, essential for high availability, will deploy with a **minimum of three brokers across different Kubernetes nodes**. This architecture ensures robustness and scalability in handling data streams.

### Volume of trade requests such as 100K in an hour
Handling a high volume of trade requests, such as 100K in an hour, requires a scalable and efficient system architecture. Here's how to process such requests effectively.

- **Optimized System Architecture**: Design a distributed and scalable system architecture using microservices, containers, and orchestration tools like Kubernetes. Distribute workload across multiple nodes to handle the high volume efficiently.
- **Asynchronous Processing**: Implement asynchronous processing to handle requests concurrently. Utilize message queues like Kafka or RabbitMQ to buffer incoming requests and process them asynchronously, decoupling request processing from response generation.
- **Horizontal Scaling**: Scale out your application horizontally by adding more instances or containers to handle increased load. Autoscaling mechanisms can automatically adjust resources based on demand, ensuring optimal performance during peak hours.
- **Optimized Database Access**: Optimize database access patterns to handle high concurrency. Utilize database sharding, replication, and indexing techniques to distribute the load and improve database performance.
- **Rate Limiting and Throttling**: Implement rate limiting and request throttling mechanisms to control the rate of incoming requests and prevent overload on the system. Define thresholds and limits to ensure fair usage and protect against abuse or denial-of-service attacks.
- **Monitoring and Alerting**: Set up comprehensive monitoring and alerting systems to track system performance, resource utilization, and response times. Use tools like Prometheus, Grafana, or Datadog to monitor key metrics and receive alerts for potential issue.
### Deployment strategy
For deploying a microservices architecture and Kafka brokers, a comprehensive deployment strategy is essential to ensure scalability, reliability, and maintainability. Here's a deployment strategy tailored for both microservices.

- **Deployment Automation**: Automate deployment processes using Continuous Integration/Continuous Deployment (CI/CD) pipelines. Tools like Azure DevOps, Jenkins, GitLab CI/CD, or CircleCI can automate building, testing, and deploying microservices.
- **Code Qaluality and Vulnerabilities**: We will integrate various tools like MEND and SonarQube for a better code quality and handling the Vulnerabilties in our build pipelines.
- **Unit and Integration Tests**: we will make sure that before commiting the code in master we should have a code coverage of 80%. So we will enforce Unit and Integration test cases.
- **Enviorment specific Build & Release Pipeline**: We will different pipeline for build and release like Dev, QA, UAT and Production. We will have auotdeployment of the build in release once the code is committed in the git. But for higher enviroment we will follow the pre-defined release cycle and would be controlled with required authorization.
- **Blue-Green Deployment vs Canary Deployment**:  We will decided based upon the application criticality that we would follow Blue-Green (two identical production environments, often referred to as Blue and Green, are maintained. At any given time, only one of these environments serves live traffic, while the other remains idle) or Canary Deployment(nvolves gradually rolling out a new version of the application to a subset of users or servers, often referred to as "canaries," while the majority of users continue to use the stable version.) in UAT/production.
### Database and why?
-**No-SQL**- For OrderRequest and OrderStatus, where flexible structure and frequent reads and writes are crucial, NoSQL databases like MongoDB offer ideal horizontal scaling capabilities. With order_id as the partition key, MongoDB ensures optimized responses, making it suitable for high-performance requirements.
-SQL- For managing Wallet and Holding information as a Transaction statement, SQL databases like Oracle or MySQL are suitable choices. Scaling may not be a major concern, but utilizing concepts like **Partitions and Sharding** ensures optimized response times. These techniques help distribute and manage data efficiently, enhancing performance without the need for drastic scaling measures.

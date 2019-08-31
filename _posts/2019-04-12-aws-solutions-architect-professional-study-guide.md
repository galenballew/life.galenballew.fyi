---
title: 'AWS Solutions Architect Professional Study Guide'
date: 2019-02-24 00:00:00
excerpt: A study guide based on the new 2019 exam guide. 
categories: [cloud]
tags: [aws, s3, certification, cloudfront, cicd]
featured_image: '/images/demo/demo-portrait.jpg'
---
# Abstract

These are my raw format notes that I took while studying for the [AWS Solutions Architect - Professional](https://aws.amazon.com/certification/certified-solutions-architect-professional/) certifcation. I passed the test and am [officially certified!](https://www.linkedin.com/in/galenballew/) 

---

Domain 1: Design for Organizational Complexity 12.5%  
1.1. Determine cross-account authentication and access strategy for complex organizations (for example, an
organization with varying compliance requirements, multiple business units, and varying scalability
requirements).  
1.2. Determine how to design networks for complex organizations (for example, an organization with varying
compliance requirements, multiple business units, and varying scalability requirements).  
1.3. Determine how to design a multi-account AWS environment for complex organizations (for example, an
organization with varying compliance requirements, multiple business units, and varying scalability
requirements).  

- Prohibit the use of unapproved services in production AWS accounts and minimize additional management overhead as the number of accounts increases
  - AWS Organizations
    - consolidated billing or "all features"?
      - Provides single payer and centralized cost tracking
        - both
      - Lets you create and invite accounts
        - both
      - Allows you to apply policy-based controls 
        - all features only
        - service control policies
          - places onto Organizational Units (OUs) within AWS Organizations
      - Helps you simplify organization-wide management of AWS services
        - all features only

- Different teams all have their own non-production accounts with AWS Organizations. Constraints need to be put in place to control costs without affecting IAM permissions.
  - Where should the AWS Budget be created?
    - master account or development account?
      - master account can create budgets and alerting for linked accounts
  - forecasted budget reaches 100% triggers a Lambda. Lambda applies a SCP to deny new infrastructure

- VPC A is peered with VPC B and VPC C. VPC B and VPC C have matching CIDR blocks, and their subnets have matching CIDR blocks. The route table for subnet B in VPC B points to the VPC peering connection pcx-aaaabbbb to access the VPC A subnet. The VPC A route table is configured to send 10.0.0.0/16 traffic to peering connection pcx-aaaaccccc.

| Route Table       | Destination   | Target       |   |   |
|-------------------|---------------|--------------|---|---|
| Subnet B in VPC B | 10.0.0.0/16   | Local        |   |   |
|                   | 172.16.0.0/24 | pcx-aaaabbbb |   |   |
| VPC A             | 172.16.0.0/24 | Local        |   |   |
|                   | 10.0.0.0/16   | pcx-aaaacccc |   |   |

  - AWS currently does not support unicast reverse path forwarding in VPC peering connections that checks the source IP of packets and routes reply packets back to the source.
  - If subnet B in VPC C has an instance with an IP address of 10.0.1.66/32, it receives the response traffic from VPC A. The instance in subnet B in VPC B does not receive a response to its request to VPC A because VPC A will route the response to VPC C.
  - To prevent this, you can add a specific route to VPC A's route table with a destination of 10.0.1.0/24 and a target of pcx-aaaabbbb. The route for 10.0.1.0/24 traffic is more specific, therefore traffic destined for the 10.0.1.0/24 IP address range goes via VPC peering connection pcx-aaaabbbb
  - The key take away is that more specific routes (meaning they map to smaller CIDR ranges) will always take precedence.

---


Domain 2: Design for New Solutions 31%  
2.1. Determine security requirements and controls when designing and implementing a solution.  
2.2. Determine a solution design and implementation strategy to meet reliability requirements.  
2.3. Determine a solution design to ensure business continuity.  
2.4. Determine a solution design to meet performance objectives.  
2.5. Determine a deployment strategy to meet business requirements when designing and implementing a
solution.  


- you are designing an application that will perform several calculations each night. the calculations are independent of each other and may take several hours to complete. what design will minimize costs, minimize interdependencies, and execute the calculations in parallel?
  - AWS Batch
    - integrates with EC2, Spot Fleet, and ECS

- you are designing an automated DR plan for a multi-tier web application. it must failover to a second region with an RTO of 30 minutes and an RPO of 15 minutes. there is a health check alarm that publishes an SNS notification if the application fails.
  - AWS Lambda subscriptions to SNS
    - change the desired capacity of your pilot light auto scaling group in the 2nd region.
    - promote the RDS read replica in the second region to a standalone DB instance.

- you want to monitor and analyze network traffic for possible threats. what solution requires minimal development and administration, scales to accommodate large amounts of network traffic, and allows queries and visualizations of the data?
  - VPC flow logs
    - all traffic will be published to CloudWatch log stream. but it needs to be archived in S3 in for better flexibility and control over log retention and the analysis.
  - Kinesis Data Firehose can push the logs to S3. Kinesis is great because it will happen in near real-time versus a scheduled migration. 
  - Athena can create an external table and query the logs using SQL syntax.
  - QuickSight can connect to Athena to visualize the data.

- your application uses a log stream. each record in the stream may contain up to 400 KB od data. design a solution to caputure a subset of metrics from the stream to be analyzed for trends over time using complex SQL queries.
  - Kinesis Data Streams versus Kinesis Firehose
    - There are two major differences:
      - Streams requires capacity provisioning. You must select a number of shards (i.e., ingestion throughput), whereas Firehose will scale automatically. 
      - Firehose can only stream into S3, Elasticsearch, and Redshift. Kinesis Data Streams can be leveraged to stream into any custom application.
    - Kinesis Analytics
      - With Amazon Kinesis Data Analytics for SQL Applications, you can process and analyze streaming data using standard SQL. The service enables you to quickly author and run powerful SQL code against streaming sources to perform time series analytics, feed real-time dashboards, and create real-time metrics.

- you're deploying a database server cluster that requires minimum network latency between nodes and maximum network throughput. What features in EC2 will be useful?
  - Enhanced Networking
    - Elastic Network Adapter (ENA)
      - up to 100 GB for supported EC2 instance types
    - Intel 82599 Virtual Function (VF) interface
      - up to 10 GB for supported EC2 instance types
  - Jumbo Frames
    - The maximum transmission unit (MTU) of a network connection is the size, in bytes, of the largest permissible packet that can be passed over the connection. The larger the MTU of a connection, the more data that can be passed in a single packet.
    - Jumbo frames allow more than 1500 bytes of data by increasing the payload size per packet, and thus increasing the percentage of the packet that is not packet overhead. Fewer packets are needed to send the same amount of usable data. 
    - For instances that are collocated inside a cluster placement group, jumbo frames help to achieve the maximum network throughput possible, and they are recommended in this case. 
  - Placement Groups
    - cluster
      - all instances are within the same AZ and can span peered VPCs
      - The chief benefit of a cluster placement group, in addition to a 10 Gbps flow limit, is the non-blocking, non-oversubscribed, fully bi-sectional nature of the connectivity. In other words, all nodes within the placement group can talk to all other nodes within the placement group at the full line rate of 10 Gbps flows and 25 aggregate without any slowing due to over-subscription.
    - spread
      - A spread placement group is a group of instances that are each placed on distinct racks, with each rack having its own network and power source.
      - multi-az
      - max of 7 instances
    - partition
      - similar to spread, but instances are grouped into partitions rather than each individual instance running on its own rack
      - max of 7 partitions

- you are migrating an application that uses an API key which is stored in a local file. after the migration, the application will be run on EC2 instances and the API key must be more secure. The API key should be unique for each environment, access requests should be auditable, it should be encrypted at rest, and access permissions should be granular. 
  - AWS Systems Manager Parameter Store + KMS
    - SecureString
      - uses default aws/ssm KMS key or a specified key
    - SSM permissions are distinct from key policy permissions
    - All API calls will be recorded in CloudTrail

---

Domain 3: Migration Planning 15%  
3.1. Select existing workloads and processes for potential migration to the cloud.  
3.2. Select migration tools and/or services for new and migrated solutions based on detailed AWS knowledge.  
3.3. Determine a new cloud architecture for an existing solution.  
3.4. Determine a strategy for migrating existing on-premises workloads to the cloud.  

- You are migrating a MySQL database. The database is forecasted to grow and the company wants to reduce the administrative/operational burden of maintaining it. What is a cost-effective solution that can be implemented quickly, requires minimal administration, and offers high performance now and in the future? Must be HA and remain operational during the migration.
  - Aurora vs RDS
    - Aurora is a fully managed relational *database engine* that's compatible with MySQL and PostgreSQL.
    - RDS is a web service that makes it easier to set up, operate, and scale a relational database in the cloud.
      - Supports MariaDB, Microsoft SQL Server, MySQL, Oracle, and PostgreSQL
    - Aurora will automatically scale storage - you do not need to plan for capacity like you do with RDS.
    - Aurora can provision up to 15 replicas. RDS MySQL can have 5.
    - Failover on Aurora is automatic. Failover is a manual process on RDS.
  - AWS Database Migration Service (AWS DMS)
    - At its most basic level, AWS DMS is a server in the AWS Cloud that runs replication software. You create a source and target connection to tell AWS DMS where to extract from and load to. Then you schedule a task that runs on this server to move your data.
    - With AWS DMS, you can perform one-time migrations, and you can replicate ongoing changes to keep sources and targets in sync.
    - If you want to change database engines, you can use the AWS Schema Conversion Tool (AWS SCT) to translate your database schema to the new platform.

- You must regularly transfer data from on-prem to S3. Data must be encrypted in-transit and at-rest. Data cannot traverse the public internet. The bucket can only be accessed from on-prem network and VPC.
  - Direct connect with private VIF
  - Create an S3 VPC endpoint. This prevents S3 traffic from going to the usual, public endpoint over the internet. 
  - Configure bucket policy to deny non-VPC traffic and to require TLS/SSL. 
  - use load balanced ec2 to proxy traffic from on-prem to the s3 vpc endpoint

- a company wants to migrate a multi-tier web app with minimal changes to the code, cost, and platform management. the stack consists of a hardware load balancer, 2 apache hosts, 4 Java/Tomcat application servers, and a MySQL server.
  - 
  
---

Domain 4: Cost Control 12.5%  
4.1. Select a cost-effective pricing model for a solution.  
4.2. Determine which controls to design and implement that will ensure cost optimization.  
4.3. Identify opportunities to reduce cost in an existing solution.  

- an application receives data every hour, on the hour. the data is processed for 50 minutes and produces a 10GB output. This output is heavily accessed during the first hour it is available with useage dropping as new outputs become available. what's the **MOST** cost-effective architecture?
    - Absolute cheapest way for compute would be EC2 1-hour Spot blocks in multiple regions
    - Cheapest storage class would be S3 One Zone-Infrequent Access with a Lifecycle policy to transition to Glacier or S3 Deep Glacier Archive

- A company is migrating all of its data to S3 within the next 4 weeks. There is 900 TB and a 100 Mbps internet link. Up to 20% of the throughput is regularly used by existing systems. Whats the **MOST** cost-effective way to migrate the data in time? 
  - Order multiple Snowballs

---

Domain 5: Continuous Improvement for Existing Solutions 29%  
5.1. Troubleshoot solution architectures.   
5.2. Determine a strategy to improve an existing solution for operational excellence.  
5.3. Determine a strategy to improve the reliability of an existing solution.  
5.4. Determine a strategy to improve the performance of an existing solution.  
5.5. Determine a strategy to improve the security of an existing solution.  
5.6. Determine how to improve the deployment of an existing solution.  

- Your application accesses a on-premises database over a 1 Gbps AWS Direct Connect link. The database is also used by applications in the data center. What can you do to make your architecture better?
  - Direct Connect is a single point of failure. If you do not have multiple Direct Connect connections, you should also establish a VPN connection between the data center and the virtual private gateway in the VPC.

- Your application sends logs to CloudWatch and also has a DynamoDB table to record the number of times different operations are invoked. The number of GET invocations are much higher in the logs than what is recorded in the DynamoDB table. 
  - Use conditional writes when the UpdateItem call saves the incremented amount.

- an application uploads files greater than 10 GB to S3, but far away locations have performance issues. 
  - Enable S3 transfer acceleration on the bucket
  - Use multi-part upload
  - **_do not_** worry about random prefixes. This is no longer a performance bottleneck in S3 for reads or writes.

- An application runs its database on an RDS instance. During peak usage periods, some user requests time out. The CloudWatch DiskQueueDepth metric spikes during these periods. How can you improve the application?
  - Modify RDS instance storage type to Provisioned IOPS SSD (io1) and provision maximum IOPS.

- Your web app sits behind an ELB ALB. There have been spikes in traffic that cause the app to slow down and fail. Logs reveal the additional traffic contained malformed request from multiple sources. What solution will **MOST** *quickly* block these types of attacks? AKA "how to minimize the impact of a DDoS attack?"
  - AWS Shield
    - AWS Shield is a managed DDoS protection service that is available in two tiers: Standard and Advanced. AWS Shield Standard applies always-on detection and inline mitigation techniques, such as deterministic packet filtering and priority-based traffic shaping, to minimize application downtime and latency. AWS Shield Standard is included automatically and transparently to your Elastic Load Balancing load balancers, Amazon CloudFront distributions, and Amazon Route 53 resources at no additional cost. 
  - AWS WAF
    - AWS WAF is a web application firewall that helps protect web applications from common web exploits that could affect application availability, compromise security, or consume excessive resources. You can use AWS WAF to define customizable web security rules that control which traffic accesses your web applications. If you use AWS Shield Advanced, you can use AWS WAF at no extra cost for those protected resources and can engage the DRT to create WAF rules.
  - Route 53
    - One of the most common targets of DDoS attacks is the Domain Name System (DNS). Amazon Route 53 is a highly available and scalable DNS service designed to route end users to infrastructure running inside or outside of AWS. Route 53 makes it possible to manage traffic globally through a variety of routing types, and provides out-of-the-box shuffle sharding and Anycast routing capabilities to protect domain names from DNS-based DDoS attacks.
  - CloudFront
    - Amazon CloudFront distributes traffic across multiple edge locations and filters requests to ensure that only valid HTTP(S) requests will be forwarded to backend hosts. CloudFront also supports geoblocking, which you can use to prevent requests from particular geographic locations from being served.
  - ELB
    - Elastic Load Balancing, like CloudFront, only supports valid TCP requests, so DDoS attacks such as UDP and SYN floods are not able to reach EC2 instances. 
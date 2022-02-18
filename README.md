# The e-commerce architecture for Octank based on microservice
It is an assignment as a part of aws application. I will share this in order to help someone who want to appliy Amazon Web Service as a solutions architect. 

## The e-commerce architecture for Octank based on microservice

### Introduction

This document proposes an effective architecture of e-commerce applications for Octank. Also, it describes a robust way of migration from on-premises to cloud infrastructure where the migration uses minimum downtime and reduces the risk of rollback. We assume the e-commerce system requires several functional parts represented as catalog, order, and account. These parts can be designed by a microservice since it is a critical part for an e-commerce system and loosely coupled to each other.

● Catalog: a catalog of e-commerce showcases products where web users should be able to select one from a list of products and compare prices. Thus, catalog requires to manage image / video and detail descriptions since a product can be represented as a picture, video, thumbnail and text descript ion.

● Order: a selected item by the user will be put into a shopping cart and then ordered. In order to make sure the order, a notification by SMS or email are required in the order service. 

● Account: all users are required to register in the system and the user profile should be managed to provide enhanced shopping information.
There are more parts such as customer service to take care of customer complaints and opinions, push notification to promote special offers, and shipment tracking required interworking with shipping companies to discover delivery status. But these are not covered in this proposal since this document is focusing on the microservice architecture and the migration from on-premises to cloud infrastructure. 


### The basic architecture
Figure 1 shows the proposed architecture for e-commerce business where users access the web site by a web browser or dedicated client. The architecture is highly available to resist the failure of a single component and highly scalable for traffic management. Also, it is cost effective and easily migrates from old monolithic architecture to a cloud infrastructure based on microservice.  
One important thing to design a system is that developers are able to make prototypes quickly and build sophisticated applications with minimal effort. Fast development and deployment are important to reduce the period of innovation so that products may quickly curate and present with trendy fashion.
Since the quality of product image / video and the speed of rendering contents are important for e-commerce business, the web contents such as HTML, CSS, JavaScript, image, and video will be loaded from Amazon CloudFront which is a famous CDN(Content Delivery Network) provided by Amazon where CDN enhances load time of content using edge network and cashing. Amazon CloudFront provides high availability and data durability for hosting and can be used to combat DDOS(Distributed Denial of Service) or distributed denial of service attacks by filtering malformed requests. We will use Amazon S3(Simple Storage Service) to store original contents of cloudFront since Amazon S3 has a comparative price policy to store and retrieve contents. 
The microservices of catalog, order, and account should support RESTful APIs for clients. Also, since the target address of a monolithic service is usually a specific domain address like www.octank.com, a path routing is required as www.octank.com/get_item. Also, load balancing is required for microservices to distribute incoming traffic across EC2 instances. Thus, we will use Amazon ALB (Application Load Balancer) for path routing and load balancing based on Layer 7. Also Amazaon ALB ables to route traffic to health targets from registered at scale.

Figure 1 the proposed architecture for e-commerce

The account service uses a database to verify a user and manage user profiles about name, age, sex, and address. Amazon Aurora is a fully managed relational database and compatible with MySQL and PostgreSQL. Also, it supports database clustering and replication. So, Amazon Aurora will be used for the account database. 
Since a product is defined as several attributes such as product id, name, price, provider, image, and detain description, document type can be applied to store a product where json or xml type is usually used. Thus, Amazon DynamoDB is selected for catalog service where DynamoDB is a famous NoSQL datastore and stores data persistently with key-value. 
In order to reduce the number of requests for DynamoDB, Amazon ElastiCache is integrated as shown in Figure 1. Amazon ElastiCache provides high performance, resizable and cost effective in-memory cache which removes the complexity associated with deploying and managing a distributed cache environment. Also, we will use ElastiCache for redis. 
Notifications of buying, selling and shipping will be sent by Amazon SNS(Simple Notification Service) which is a fully managed messaging service for SMS, email, and mobile push notifications. Also, it can send millions of promotional messages with reliable and scalable manners automatically.
Figure 2 shows an implantation of the proposed architecture based on AWS cloud infrastructure. We will use Amazon EKS(Elastic Kubernetes Service) as an orchestration tool. It provides highly available and secure clusters for microservices. VPC is configured with public and private subnets in order to manage private IPs and security. Also, NAT gateways allow outbound internet access for the microservices of catalog, order, and account from private subnet. For secured access from outside, ACL and SG should be set by the role. Also we use Amazon Route53 which is a highly available and scalable DNS(Domain Name System) web service and registers domain names..
In order to ensure high availability, the server should be set up in Multi-AZ (2 or 3 different Availability Zones) so that the service continues to work even if one of the availability zones goes down. Also, we will apply different ASG(Auto Scaling Group) for catalog, order, and account since we want to optimize EC2 node groups effectively based on the purpose of the service. 

Figure 2 AWS implementation based on microservice architecture

Inactive objects such as snapshots or logs should be capable of archiving into a cheap storage when it is old for example, greater than 6 months. We will use Amazon S3 Glacier to archive old data as a secure, durable and low-cost storage for data archiving and long-term backup.
Images or videos about products should be stored in storage after uploaded by sellers. Also, the contents will be easily uploaded to CloudFront for fast access from any place. Thus, we will use Amazon S3 for the contents since it is highly scalable, reliable, fast and inexpensive data storage. 
The encryption of data is mandatory since the security of data transmission and storage are really important for e-commerce. We will use AWS KMS(Key Management Service) in order to manage the encryption key safely and controlled by the permission for the access of the key.
In this architecture, all microservices are loosely coupled. So, we can separate the migration from on-premises to cloud infrastructure by several phases. But in this case, we may change the routing role in the on-premises server and need to take several steps to migrate databases. Usually, it requires deep understanding about business logic. Also, since the proposed architecture supports auto scaling for instances and Step-by-Step migration, we can handle the uncertainty about resources and time flexibly.


### Monitoring
The microservice architecture requires a high level of monitoring compared with a monolithic architecture since it should manage hundreds of Pods and different types of instances continuously. Also, we should check the operation of EKS carefully.
Amazon ElasticSearch Service is a cost-effective managed service for log analytics and full-text search. Also, the managed Kibana included in Amazon ElasticSearch is a popular and powerful visualization tool based on open source in ElasticSearch. In this proposal, the log of microservices from catalog, order and account in EC2 will be collected by Kinesis Agent and then send it continuously to Amazon Kinesis Data Firehose. Finally, the indexed data by Amazon Elasticsearch will be analyzed and visualized by Kibana.
Amazon Managed Service for Prometheus(AMP) is a prometheus compatible monitoring service.  Prometheus can collect and query metrics from AWS container services including Amazon EKS. Also, Amazon Grafana is a popular open source analytics platform that enables you to query, visualize, alert on and understand your metrics no matter where they are stored.
Figure 3 shows the monitoring process for microservices where the operator can check the status of microservices using Kibana and Grafana.

Figure 3 Monitoring of microservices
 
### Migration Plan
The network connection between on-premises and cloud can use AWS Direct Connect or VPN. Amazon DMS(Database Migration Service) can migrate on-premises to Amazon Aurora or Amazon DynamoDB with minimal downtime. Also, Amazon DMS supports heterogeneous migration between different database platforms. Also, the database of on-premises can be continuously replicated with high availability.
Amazon SCT(Schema Conversion Tool) converts the existing database schema for the target database suitably. And it provides a project-based user interface to convert the database schema so that the user can easily convert it.
After the database migration, the migration of processes should be Step-by-Step Walkthrough. So, weighted routing by Route53 is used to route traffic from on-premises to cloud. 
Figure 3 shows the migration of databases from on-premises to cloud.


Figure 3 Database migration
 
### Summary
The proposed architecture is highly available and satisfies the requirements where we have applied EKS to manage the Pods dynamically so that Pods can be recovered for microservices.  The auto scaling groups in which each microservice uses their own group effectively optimizes the resource of microserves for variant traffic. 
In order to enhance the performance of throughput for databases, we applied cache logic using Amazon ElastiCache. Also, we applied NoSQL and SQL databases for catalog and account services based on the characteristics of the data attribute so that the user data can be analyzed or scaled.  
For monitoring the microservice’s operation, we use Elasticsearch and Kibana with Kinesis Data Firehose. Also, the operation of EKS is monitored by Grafana and Prometheus so that an alarm can be generated if some issues happen.
In order to migrate from on-premise to cloud, we show a method where the database is robustly moved by using SCT and DMS supported by AWS. Also, the traffic can be managed using weighted routing by Route53 for Step-by-Step migration.
 
 
 

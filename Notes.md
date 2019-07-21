## EC2Lab:

* Launch Instance
* Select "Amazon Linux AMI 2018.03.0 (HVM), SSD Volume Type" AMI
* Pick t2.micro instance for instance type
* Click **Configure Instance Details** (keep all the settings default). 
    * **Enable termination protection**: To protect instances getting terminated by accident. Good for production.
    * **Enable CloudWatch detailed monitoring**: Detail monitoring. Default monitoring checks every 5 min. Enabling would make it every minute.
    * **Advance Details -> User Data**: This is used for bootstrapped scripts. This can be left as default.
* Click **Add Storage**
    * Set your boot volume. Leave everything as default.
* Click **Add Tags**
    * You could add any tags to talk about your EC2. For example, staff id, department id, etc.
* Click **Configure Security Group**
    * Think of security group as your virtual firewall.
    * Click on "Create a new security group" and make sure your security rules looks like below
[![Security Group](https://i.imgur.com/BDB7PoF.png "Security Group")](https://i.imgur.com/BDB7PoF.png "Security Group")


Once you can log into your EC2 run following command to start a small web server:
* Become the root user: `sudo su`
* Update the environemnt: `yum update -y`
* Install Apache web server: `yum install httpd -y`
* Start the service: `service httpd start`
* When this EC2 reboots, we want it to automatically start this server. `chkconfig httpd on`
* Check status: `service httpd status`
* Apcahe created a directory `/var/www/html/`. This should be empty. `cd` into it.
* Create a file called `index.html` with following content:
```html
<html>
	<body>
		<h1>Hello World!</h1>
	</body>
</html>
```
* Using the web browser, hit the public ip of this instance. It should display **Hello World!** page.

## Elastic Load Balance
* 504 Error means that application has timed out. This means the application has failed to respond within idle timeout period. Is the web server down? Database down? Scale up/out?
* When user makes a request to the application, ELB routes the request to the appropriate EC2 running your application. This enabled the application to only see ELB privates address. However . . .
* X-Forwarded-For header of the request, supplied by ELB, allows you to see user's public ip (IPv4).
* ELB are put in front of a _target group_. Target Group is a set of aws resources like that ELB will route requests to (depending on your requests).

### Create an ELB Application Load Balancer

1. Go to EC2.
2. Go to "Load Balancers" on the left pane.
3. Click on Application Load Balancer.
4. Internet-Facing, IPv4 address type.
5. Application Load Balancer is going to listen on HTTP on port 80.
6. Select all availability zone to give maximum redundency. 
7. Then assign a security group. Go ahead and assign the security group we created in the last lecture.
8. Now, configure routing

![image-20190519161857917](/Users/kgupta/projects/aws-certified-developer-associate-course/image-20190519161857917.png)

9. Review and Create.
10. It will take some time to provision.

In the above picture `Path` is going to be used to do health checks. In this case, it will check if `/index.html` exists or not. If it exists, health is good.

#### Verification

To check if your creation of ELB went good. Follow following steps:

1. Go to **Target Groups** under **EC2** service.

   ![image-20190519162536972](/Users/kgupta/projects/aws-certified-developer-associate-course/image-20190519162536972.png)

2. Find your Target Group.

![image-20190519162945752](/Users/kgupta/projects/aws-certified-developer-associate-course/image-20190519162945752.png)

3. Click Target and check the status on the right hand side.

![image-20190519163028417](/Users/kgupta/projects/aws-certified-developer-associate-course/image-20190519163028417.png)

## Route53

Route53 is amazon DNS service that allows you to map domain names to following three different backends:
* EC2 Instances
* Load Balancers
* S3 Buckets

We need to create a Hosted Zone. 
1. However, before you create the hosted zone you need to register a domain. (It takes 3 days to complete the process. Only then you can proceed.)
2. Once registered and processed, go back to **Hosted Zone** display. You should see your domain there.
3. Click on "Create Record Set". A record set is a dictionary that lets your domain name get mapped to a specific backend. The connection is made by an IP address.
4. Find your Application Load Balancer you created and assign that to the domain like below

![image-20190519163330066](/Users/kgupta/projects/aws-certified-developer-associate-course/image-20190519163330066.png)

#### Verification

To verify that you can use the domain and hit the website you created just simply go to the web browser and type in your domain address. For example, here `deprosun.com`

![image-20190519163615333](/Users/kgupta/projects/aws-certified-developer-associate-course/image-20190519163615333.png)

# Lambda

Lambda is a compute service that allows you to upload your code and create a Lambda function.  Lambda remove the stress of following:

* Data Centers

* Hardware

* Assembly Code/Protocols

* High Level Languages

* Operating Systems

* Application Layer/AWS API

  

###Lambda Use Cases

* Lambdas can be event driven where, for example, the Lambda will be kicked off if something changes in S3.

* Lambdas can be used as compute service where it runs your code in response to HTTP requests using Amazon API Gateway or API calls using AWS SDKs.

  

### Lambda Languages

* Node.js

* Java

* Python

* C#

* Go

  

### Lambda Exam Tips

* Lambda scales out (not up) automatically 
* Lambda functions are independent, 1 event = 1 function
* Lambda is serverless
* Lambda functions can trigger other Lambda functions. 1 event can kick x functions if functions trigger other functions.
  * Architecture can get complicated. AWS X-Ray allows you to debug what is happening.
  
  * Lambda can go things globally. You can backup S3 buckets to other S3 buckets etc.
  
    

### Lambda Triggers

* API Gateway

* AWS IoT

* Alexa Skills Kit

* Alexa Smart Home

* Application Load Balancer

* CloudFront

* CloudWatch Events

* CloudWatch Logs

* Cognito Sync Trigger

* DynamoDB

* Kinesis

* S3

* SNS

* SQS

  

### Lambda Version Control

* Can have multiple version of lambda functions
* Latest version will use $latest
* Qualified version will use $latest, unqualified version will not have it
* Versions are immutable
* Can split traffic between aliases to different versions
  
  * Cannot split traffic with $latest, instead create an alias to latest
  
    

## API Gateway

* API Gateway has caching capabilties to increase performance

* API Gateway is low cost and scales automatically

* You can throttle API Gateway to prevent attacks

* You can log results to CloudWatch

* If you are using Javascript/AJAX that uses multiple domains with API Gateway, ensure that you have enabled CORS on API Gateway 

* CORS is enforced by the client

  

## Step Functions

* Great way to visualize your serveless application

* Step functions automatically trigger and tracks each step

* Step functions logs the state of each step so if something goes wrong you can track what went wrong and where

  

## S3

* Files can be from 0 Bytes to 5 TB

* Unlimited storage

* S3 is univeral namespace. Bucket names have to be unique across the globe

* Read after Write consistency for PUTS of new objects

* Eventual Consistency for overwrites PUTS and DELETES (can take some time to propogate)

* Core fundamentals of an S3 Object:
  * Key (name of the file)
  * Value (data in the file)
  * Version ID
  * Metadata (it can user defined tags)
  * Subresources - bucket specific configuration:
    * Bucket Policies, Access Control Lists
    * Cross origin Resource Sharing (CORS)
    * Transfer Acceleration
  
* Read S3 FQA at https://aws.amazon.com/s3/faqs/

  

### Storage Tier / Classes

* S3: 99.99% availability, 99.99% durability, stored redundantly across multiple devices in multiple facilities and is designed to sustain the loss of 2 facilities concurrently.
* S3 - IA (Infrequently Accessed) : For data that is accessed less frequently but requires rapid access when needed. Lower fees than S3, but you are charged a retrieval fee.
* S3 - One Zone IA : Same as IA however data is stored in a single Availability Zone only. Still 99.99% durability, but only 99.5% availability. Cost in 20% less than regular S3 - IA.
* Reduced Redundancy Storage: Designed to provide 99.99% durability and 99.99% availability of objects over a given year. Used for data that can be recreated if lost, e.g. thumbnails. (AWS not recommends this anymore..)
* Glacier: Very cheap but used for archival only. Optimized for data that is infrequently accessed.
  * Takes 3 - 5 hours to restore fro Glacier.

![image-20190608111237471](/Users/kgupta/projects/aws-certified-developer-associate-course/image-20190608111237471.png)



### S3 - Intelligent Tiering

This is for objects that have unpredictable access patterns.  It's comprised of two tiers within in it:

* Frequent Access
* Infrequent Access

It automatically moves the object to most cost-effective tier based on access frequency. There is no fees for accessing your data  but a small monthly fee for monitoring/automation $0.0025 per 1,000 objects.



## S3 Charges

Charged for:

* Storage per GB
* Requests (Get, Put, Copy, etc)
* Storage Management Pricing
  
  * Inventory, Analytics, and Object Tags (Object tags are like "what project or team they relate to or something")
* Data Management Pricing
  
  * Data transferred out of S3
* Transfer Acceleration
  
  * Use CloudFront to Optimize transfers
  
    

## CloudFront

* Edge Location: This is where the contents are cached. This is separate from AWS Region and Availability Zone. 
  * Edge locations are not read only - you can WRITE to them too.
  * CloudFront Edge Locations are utilized  by S3 Transfer Acceleration to reduce latency for S3 uploads.
  * Objects are cached for the life of the TTL (Time To Live)
  * You can clear cached objects but you will be charged.
* Origin: This is the origin of all the files that the CDN will distribute. Origins can be an S3 bucket, an EC2 instance, an Elastic Load Balancer, or Route53.
* Distribution: This is the name given the CDN, which consists of a collection of Edge Locations. There are two types of distributions
  * Web Distributions: for the websites
  
  * RTMP Distributions: Used for Media Streaming
  
    

## Simple Queue Service (SQS)

* SQS is pull based, not push based.

* Messages are 256KB in size

* Messages can be kept in the queue from 1 min to 14 days

* Default retention period is 4 days

* SQS guarantees that your messages will be processed at least once

* Visibilty Timeout
  * Messages in the queue can be invisible for upto 12 hours.
  * Default is 30 seconds
  
* Short polling: returned immediately even if no messages are in the queue

* Long Polling: polls the queue periodically and not only returns a response when a message is in the queue or a timeout is reached

  

## Simple Notification Service (SNS)

* SNS is a scalable and highly available notification service which allows you to send push notifications from the cloud.

* Variety of message formats supported: SMS Text Message, Email, Amazon Simple Queue Service (SQS) queues, any HTTP endpoint

* Pub-Sub model where user subscribe to topics.

* It is a pull mechanism, rather than pull (poll) mechanism

  

## Simple Email Service (SES)

* It an email service.

* Not pub-sub, you just need to know the email address

* For marketing emails, recipt conformation, etc.

  

## SNS Vs SES

* SES is for email only
* It can be used for incoming and outgoing emails
* It is not subscription based, you only need to know the email address
* **SNS** supports multiple formats (SMS, SQS, HTTP, Email)
* Push notifications only
* Pub/Sub model: consumers must subscribe to a topic
* You can **fan-out** messages to large number of recepients (e.g. multiple clients each with their SQS queue)



## Elastic Beanstalk

* Deploys and scales your web applications including the web application server platform where required 
* Supports widely used programming technologies - JAVA, Python, Ruby, Go, Docker, .NET, Node.js
* And application server platform like Tomcat, Passenger, PUMA, and IIS
* Provision underlying resource for you
* Can fully mannage the EC2 instances for you or you can take full administrative control
* Updates, monitoring, metrics and health checks all included
* Remember 4 different deployment approaches:
  * All at Once
    * Service interruption while you update the entire environment at once
    * To roll back, perform a further all at **All at Once** upgrade
  * Rolling
    * Reduced capacity during deployment
    * To roll back, perform a further rolling update
  * Rolling with addtional batch
    * Mantains full capacity
    * To roll back, perform a further rolling update
  * Immutable
    * Preferred option for mission critical production systems
    
    * Maintains full capacity
    
    * To roll back, just delete the new instances and autoscaling group
    
      

## Elastic Beanstalk Advanced

* You can customize your elastic beanstalk environment by adding configuration files

* The files are written in JSON or YAML

* Files have `.config` extension

* The `.config` files are saved inside `.ebextensions` folder

* Your `.ebextensions` folder must be located in the top level directory of your application source code bundle

  

## RDS & Elastic Beanstalk

* Two different options for launching RDS instance:

* **Launch with Elastic Beanstalk**

  * When you terminate the Elastic Beanstalk environment, the database will also be terminated
  * Quick and easy to add your database and get started
  * Suitable for Dev and Test environemnts only

* **Launch outside of Elastic Beanstalk**

  * Additional Configuration steps required - Security Group and Connection Information

  * Suitable for Production environment, more flexibility

  * Allows connections from multiple environments, you can tear down the application stack without impacting the database

    

## Kinesis

* Know the difference between Kinesis Streams and Kinesis Firehose

* **Kinesis Stream**

  * Messages is stored/sent-to shards.
  * Messages can be rentained upto 7 days. Default 24 hours.
  * Video Streams: securely stream videos from connected devices to AWS for analytics andf machine learning
  * Data Streams: Build custom applications process data in real-time
  * You can configure a Lambda to subscribe to a Kinesis Stream and execute a function on your behalf when a new record is detected, before sending the process data to its final destination

* **Kinesis Firehose**

  * No shards.
  * Automatically analyzing data using Lambdas.  Not having to worry about data consumers
  * Capture, transform, load data streams into AWS data stores for near real-time analytics BI tools

* Understand what Kinesis Analytics is.

  

## Dynamo DB

* Amazon DynamoDB is a low latency NoSQL database
* Consists of Table, Items, and Attributes
* Supports both document and key-value data models
* Supported document format are JSON, HTML, and XML
* There are two types of primary key - Partition Key and combination of Partition Key + Sort Key
* 2 Consistency Models: Strongly Consistent and Eventually Consistent
* Access is controlled using IAM Policies
* Fine grain access control using IAM Condition paramter: `dynamodb:LeadingKeys` to allow users to access only itmes where the partition key value matches their USER ID
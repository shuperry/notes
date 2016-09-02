# General
* AZ: Availability Zone
    * single data center
    * AZs are 5-15 miles apart
* Region: physical location where there are multiple AZ
    * A region is a cluster of AZs.
    * Multi-region architecture is complex and only used by very large
      applications
* Consolidated Billing
    * A large AWS user might have different AWS accounts and their bill can be
      consolidated.
    * Limited to **20** accounts
    * Reduces cost as the prices goes down as the usage goes up
* Premium support
    * Basic
    * Developer
    * Business
    * Enterprise
* Hypervisor is Xen
* 11 Regions as of now
* PCI Data Security Standards 3.2 Level 1
    * Payment Card Industry Data Security Standard (PCI DSS) 


# AWS Services
* Over 40 services but not all of them are essential
* Essential ones include:
    * EC2
    * S3
    * CloudFront
    * RDS
    * DynamoDB
    * ElastiCache
    * VPC
    * Route 53
    * SES

![list of core services](https://i.imgur.com/tOeORes.png)


# Jargons
* AMI: Amazon Machine Image. E.g Ubuntu 14.04 AMI
* ARN: Amazon Resource Name
* AZ: Availability Zone, basically a data center
* EBS: Elastic Block Store
* VPC: Virtual Private Cloud
* ASG: Automatic Scaling Group
* ELB: Elastic Load Balancer
* HA: High Availability
* ECS: EC2 Container Service
* IAM: Identify and Access Management
* EIP: Elastic IP
* EMR: Elastic MapReduce
* Data Pipeline
* SWF: Simple Workflow Service
* ECR: Elastic Container Registry?


# EC2
* Elastic Compute Cloud
* Recently started to support Docker
* Reside in a VPC (basically a private network)
* Sit behind firewall (Security Group)
* By default has a private IP.
    * Can have a public Elastic IP Address
* All use ECC memory
* Amazon Linux AMI image has a lot of built-in tools
    * Use `yum` so probably based on Redhat
* Volume is basically attached storage
* Snapshot can be created for a volume
    * When it's detached from the instance
    * new volumes can be created from that snapshot
    * snapshot can be turned into image (AMI)
* bootstrap script
    * like the `fab bootstrap` command I had but built-in into AWS
    * **Advanced Detail** section when EC2 instance is created
* There is a soft limit of **20** instances per region
    * can be lifted on request
* Must select an IAM role while creating
    * one and one only
* You are limited to **5** Elastic IP per region
    * if the IP is not attached to an instance, it'll be charged
* VM images can be imported/exported
* AMI can use two different kind of virtualisation
    * Paravirtual (PV)
    * Hardware Virtual Machine (HVM), more recent, supposedly more performant
* `http://169.254.169.254/latest/meta-data/`

## Options
* Free Tier
* On Demand
    * fixed rate by the hour
    * this is the normal case
* Reserved
    * reserved capacity for a longer time
    * cheaper
    * reserved instance can be transferred into different AZ
* Spot
    * bid whatever price for unused instance capacity

## Storage
* can use instance local storage or EBS
    * EBS volumes can be used to create RAID
* instance local will not survive reboot

## Security Group
* Firewall rules that can be applied to EC2 instance

## Elastic Load Balancer
* Can be made to work across AZ
    * ELB will launch new instance in the AZ with fewest instances
* Internal balancer (internal IP)
* External balancer
* ELB carries out health check on the instances, through
    * SSL
    * HTTP(s)
    * TCP
* When using auto scaling, one doesn't have to add EC2 instances to the back
    * automatically done by auto scaling

## CLI
* Amazon Linux AMI has command line tool pre-installed

## Placement Group
* logical grouping of instances within a single AZ
* connected by **10** Gbps network
* cannot merge placement group
* cannot move existing instance into a placement group
* AWS suggest using same instance type for all instances in the group


# Route 53
* AWS DNS service
    * global, not per region
* 53 is the DNS port
* Possible to purchase domain name
* handles DNS level load balancing as well
* Supports health check so only route to healthy nodes
    * not with FTP
* Supports alias
    * not in the standard DNS protocol
* Supports wildcard entry
* Supports zone appex, or naked zone
* Multiple IP can be assigned to a single record for balancing purpose
* No default TTL
* Built on top of anycast network
    * for HA
    * answer can come from any member of an dedicated group
* Hosted zone
    * \*.amazone.com
    * each AWS account is limited to **500** hosted zones
* By default, **50** domain are supported
    * can be raised by contacting AWS support


# RDS
* Relational database as a service
* Default to **40** instances max
* Supports
    * MySQL
    * Postgres
    * Oracle
    * SQL Server
    * MariaDB
    * Aurora (AWS proprietary, massively scalable)
* With Oracle, AWS can provide a license
    * Or you can bring your own
* Supports Multi-AZ deployment
    * Master instance + standby instance in multiple AZs
    * fail over handled transparently
    * standby instance used for both fail-over and planned maintainance
    * standby instance is not read replica
    * standby will **NOT** be in the same AZ
* Backup are turned on by default
* Backup retention period can be **35** days or less
    * can only backup InnoDB engine
* DB instance can be modified
* Restoration is possible
    * to a new endpoint, basically new instance
* create instance and specify
    * db instance name
    * master user name
    * master user password
    * db name
    * This user would have full access to the specific DB.
    * availability, if global available, then it'll have a public IP
* by default, the security group only allow a certain IP to access (which one?).
  This can be fixed by changing the security group
* change code to match the configuration and everything should work
* MySQL
    * support 5.5, 5.6 and 5.7
    * InnoDB as default engine
    * access slow-query log for finding performance problems
* backup
    * supports point-in-time restoration
    * auto backup are done by AWS, snapshot are user initiated
    * stored on S3
    * on deletion of DB instance, automatic backups are removed, snapshots
      aren't
* [how to enable utf8](https://forums.aws.amazon.com/message.jspa?messageID=574515)
* You are not going to break a SQL database with your first 10 million users. Not even close.
    * From [here](http://highscalability.com/blog/2016/1/11/a-beginners-guide-to-scaling-to-11-million-users-on-amazons.html)

## Aurora
* AWS own implementation of relational DB
* MySQL compatible
* Availability
    * **2** copies of a piece of data in one AZ
    * at least across **3** AZ


# EBS
* When mounted as root device, instance has to be stopped before detaching
    * otherwise, detachment can happy any time
* Snapshots are only available to EC2 APIs
    * not with regular S3 API
* Snapshots can be taken when the volume is attached and in use
* Snapshots only captures data that has been written to EBS


# S3
* Object storage
* file size can be 1 byte to 5 TB
* in theory there is no limit to the total amount of storage
* files are stored in buckets
    * namespace is globally unique
    * no capital letter allowed
* buckets are created in a region
    * data never leaves that region unless you ask to
* default up to **100** buckets per account
* key value store
* files are default to private
* should be used for storing blob, unstructured data
* bucket URL can be path or domain:
    * `http://bucket-name.s3.amazonaws.com`
    * `http://bucket-name.s3-aws-region.amazonaws.com`
    * so name has to be a valid domain
    * `http://s3-aws-region.amazonaws.com/bucket-name`
* can host static website
    * has to be configured so
* meta data is allowed
* events can be triggered when say upload happens
    * sent to SNS or SQS
* data can be encrypted on the server with AES256
* 99.9999999% (11 9s) durability
* 99.99% availability
* Can be mounted as local file system with `s3fs`
* Access to the S3 API is governed by an **Access Key ID** and a **Secret Access
  Key**.  The access key identifies your S3 user account while the secret key is
  akin to a password and should be kept secret.

## Storage Tiers
* S3
* S3 IA (infrequently accessed)
    * cheaper
    * same availability and durability as the standard service
    * for older data, backup data, etc
    * object smaller than **128** KB will be charged
* Reduced Redundancy Storage (RRS)
    * something that is not as vital or can be regenerated
    * 99.99% availability
    * 99.99% durability
* Glacier
    * long term storage
    * delay in data availability

## Versioning
* allow recovery of older version
* disabled by default
    * cannot be disabled once enabled
* `DELETE` is only a version marker
* this is a pre-request for cross region sync

## Encryption
* Server Side Encryption
    * SSE-S3
    * SSE-KMS
    * SSE-C (key provided by customer)
* Client Side Encryption
    * encrypting before upload

## Data Consistency Model
* for `PUT` with new data, it's **read-after-write**
* for `PUT` with existing data and `DELETE`, it's eventual consistency

## Static website
* configure index document and error document


# CloudWatch
* real time monitor of EC2 instances
* detailed is at 1 min frequency
* normally is at 5 min frequency
* does **NOT** monitor memory usage


# Import and Export Disk
* physical disk data transfer
    * USB disk brought into AWS
* can import to:
    * EBS
    * S3
    * Glacier
* can only export from S3

## Snowball
* Import/Export service
* 50TB or 80TB per snowball
* Suitcase looking device
* For very large (petabytes) amount of data
* Rent based business
* 256 bit encryption
* Implied to replace **Import and Export Disk**
* Customer can request up to **50** devices
* Jumbo frames not supported

# Storage Gateway
* Hybrid Cloud Computing
* VM image downloaded
* Three types
    * Gateway Stored Volume (all data on site and synced to S3)
    * Gateway Cached Volume (all data on S3 and cached locally)
    * Gateway Virtual Tape Library (S3 as iSCSI, using Glacier)


# DynamoDB
* NoSQL
* Always on SSD
* Automatically distributed
* Not MongoDB
* Data stored on **3** replicas
* Does not support embedded data
    * say array inside a JSON object
* Data consistency model
    * eventual consistent reads. might need a second for data consistency to
      reach
    * strongly consistent reads. reads always reflects the latest
* Partition key and sort key for finding and sorting data
* Item
    * An item cannot exceed **400 kB**
    * items can have attributes
* Qeury and Scan
    * query returns subset that matches the query condition
    * scan gets all items in the table, results can be limited
    * query will stop and return if total number of items exceeds **1 MB**

## Indexes
* Global index: hash key and range key are different from that of the table
* Local index: same hash key but different range key

## Throughput 
* AWS uses **capacity units** for provisioning throughput
* [aws doc](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ProvisionedThroughput.html)


# Auto Scaling Group
* trigger can be customized
    * instance CPU load
    * app metric
* when **ASG** is terminated, instance in that group will be terminated


# Lambda
* serverless architecture
* scaling is handled by AWS itself
* code runs on
    * db modification
    * S3 modification events
    * Kinesis
    * CloudTrail
    * SNS
    * API Gateway
    * Mobile backend
    * custom events generated by application
    * etc
* Supports:
    * Python
    * Node.js
    * Java (8 compatible)
* 99.99% availability
* Lambda function
    * has 500MB scratch space to use non-persistent
    * must finish within **300s**, with **5s** as default
    * no access to the underline system
    * code stored on S3 and encrypted at rest
    * allowed to use threads, etc
* Pricing
    * 0.20 after 1 million
    * pay by duration rounded to nearest 100ms


# SES
* email sending service


# CloudFront
* AWS CDN
* Edge location - separate from region/AZ
    * read and write
* Origin (also called **definitive copy**)
    * S3 bucket
    * EC2 instance or **any** web server (for dynamic content)
    * Elastic load balancer
    * Route53
* Distribution - collection of edge location
    * web
    * RTMP (flash)
* Works with non-AWS servers
* Can specify root object for static web hosting
* ??? signed cookie?


# EFS
* Elastic File System
* Similar to NAS
* Block level storage


# Redshift
* Data warehouse used for BI
    * cheaper and faster than traditional data warehousing
* OLTP vs OLAP
    * Online Transaction Processing
    * Online Analytical Processing
* Up to **128** compute node
* Data are column oriented instead of row oriented
* No cross AZ


# VPC
* Virtual data center essentially
* A single VPC can span across multiple AZs.
    * subnet has to be in the same AZ though
* Layered networks
    * internet facing front end
    * private back end, db, app server
* Hybrid cloud
    * connection between corporate network and AWS
* Default VPC
    * automatically internet connected
    * Don't delete!!! You will have contact AWS to get it back
    * CIDR: 172.31.0.0/16
* Peering
    * connect two different VPCs directly
    * connection does **NOT** automatically carry over
    * connection can be across AWS accounts
    * limited to same region
* Limitation
    * 5 elastic IP
    * 5 internet gateway
    * 5 VPCs per region (increasable)
* Subnet are always in one AZ
* One internet gateway per VPC
* Network ACL
    * there is a default ACL for a VPC that allow everything,
      both in and out
    * Basically firewall rules that can be associated with subnet
    * stateless
* VPC size cannot be changed once configured
* One can create up to 200 subnets per VPC
* AWS reserves starting 4 IPs and last 1 IP
* Security Group
    * You can create subnet group for a specific VPC
* Internet Gateway
    * Like residential gateway that connects a VPC to internet
* Wizard options
    * with a single public subnet
    * with public and private subnets
    * with Public and Private Subnets and Hardware VPN Access
    * with a Private Subnet Only and Hardware VPN Access
* No additional charge for VPC itself


# Elastic Search
* ....


# ElastiCache
* redis
    * replication and multi-AZ turned on by default
    * can be backedup by S3
    * can be launched into VPC
    * can seed data from S3 snapshot
    * default to 1 master and 2 read replica in different AZ


# Kinesis
* Data streaming


# DMS
* Database Migration Service


# Inspector


# Directory Service


# KMS
* Key Management System


# Cloud Formation
* Templating for AWS application structure
    * EC2
    * RDS
    * etc
* Quickly replicate infrastructure
* Track changes happened in existing infrastructure
* template is specified in JSON format
    * there is a graphical editor
    * Fields include: `Type`, `Properties`, `Metadata`, `DependsOn`
* Like Elastic Beanstalk, it's free but the actual resources are charged
* Creation settings
    * rollback on failure: default to yes
    * timeout for stack creation
    * stack creation can be notified to SNS
* CloudFormation, OpsWorks and Elastic Beanstalk basically try to achieve the
  same thing but at different level of abstraction. The latter the higher
* XXX: is it possible to update a stack after creation?


# Cloud Trail


# AppStream


# Opsworks
* Chef like automating service


# Elastic Transcoder
* media transcoder in the cloud


# SWF
* Actors
    * starter
    * decider
    * activity worker
* Domain
    * a collection of related workflows


# SQS
* Simple Queue Service
* First service launched in AWS
* Message visibility timeout
    * during this period, message is **NOT** visible
    * this happens **after** an instance has acquired the message
    * to ensure that the message can be processed without duplication
    * default to **30** seconds
    * maximum **12** hours
* Message Retention
    * how long is the message kept
    * default to **4** days
* A message can be up to **256 KB** of text in any format
    * Billing happens per **64 KB**
    * Configurable
* Doesn't guarantee first in first out.
* A message can be delivered more than once
    * at least once
* Scaling can be triggered if the queue is getting too full


# SNS
* Simple Notification Service
* Messages are pushed
* pub-sub, no polling required
* SMS, email
* messages can be sent to SQS
* compatible with
    * **APNS** for both iOS and OS X
    * Baidu push
    * Google Cloud Messaging
    * Windows Push


# IAM
* manage access to AWS console
* not reagion specific
* root account: email address when signing up AWS
* MFA: multi-factor authentication
    * additional login requirement like google authenticate
    * basically scan QR code with a phone app
* Security Assertion Markup Language
* Can **fedorate** with external authentication services
    * Web Identity Fedoration
    * XXX what's supported?
* once a EC2 instance is creaed, one cannot assign IAM role to it
* default to no permission
* with IAM one can
    * create user
    * create group
    * add user to the group
* policy can be versioned
    * you can have up to 5 versions
* account setting can be used to change things like password policy
* APIs
    * `AssumeRoleWithWebIdentity`
    * `AssumeRoleWithSAML`


# Security
* The guy suggest to keep API keys, password in environment variables.

## Trusted Advisor
* Automated tool for detecting concerns in your cloud architecture
    * open ports etc
* Security Checks 
    * open port on security groups
    * MFA on root account
    * IAM use: weather non-root user is created
    * Service limit
* Other checks
    * cost optimization
    * Performance
    * Fault Tolerance


# Launch Configuration
* Goto my AMIs
* Advanced configuration allows to add user data (bootstrapping)
* No need to add storage when using customized image
* Select 2 subnet, which maps to 2 AZ for HA
* specify load-balancer
* add scaling policy
    * notification can be sent to SNS topic
* health check should be EC2
    * so that when instance is down new one will be spinned up
* each ASG has
    * group size: initial size
    * min size, max size
    * subnet
    * LB
    * scaling policy


# Direct Connect
* dedicated network connection from your premise to AWS


# Resource Group
* resources (EC2, RDS instances etc) can be grouped by filtering out
    * tag values
    * region
    * type
* useful for large organizations with complex architectures

# Simple DB
* ???


# Elastic Beanstalk
* Paas from AWS, similiar to heroku
* Up to **25** applications
* Automatically scaled
* Can be used with all possible database solutions
* Can create traditional web server or worker which carries out long running operation
* Not a chargable service
    * Customer only pays for underlying infrastructure
* Supported language
    * Python
    * Go
    * Node.js
    * Docker
    * Java
    * Ruby
    * PHP
    * .NET
* Supported Servers
    * Nginx
    * Apache
    * IIS
    * Passenger
* Version
    * You can have multple version of an app running at the same time
    * **500** versions are allowed by default


# EC2 Container Service (ECS)
* You can create your own registry
    * for pushing, pulling
* Can benefit from IAM
* task definition
    * blueprint for containers
    * looks like an orchestration tool
    * can be configured via JSON
* cluster


# Practical
* register with an existing amazon.com account
    * CN phone number can be used
* select the free EC2 plan to launch
    * create a key pair and store the private one on the local computer.
* S3 can be used with free tier
* Production RDS cannot be used with free tier
* Login to EC2
    * `chmod 400 <private-key-file>`
    * `ssh -i <private-key-file> ubunt@<public ip>`
    * AWS even remembers the private key file name you saved!
* The default security setting for EC2 and RDS are very strict. Only SSH is allowed for EC2 instance. To fix:
    * Create a security group
    * Allow all TCP from anywhere. (Dangeous but works)
* git, pip etc operations are super fast!
* `git clone bitbucket`
* install missing stuff
* `pip install -r requirements`
* `manage.py runserver -h 0.0.0.0`
* install nginx
* click on the security group of the VPC and add port 5000 to firewall rules
* browsing `http://52.88.129.236:5000/` works
* configure nginx, see below section
* install mobileweb static files.
* `tmux` was installed by default!
* `gunicorn -b :8080 wsgi:app`
* Now `http://52.88.129.236` works, not slow either
* change config and replace 192.168.1.6, etc

## WordPress
* Create security group inside default VPC
    * web service
    * RDS service (allow web service group to access 3306)
* Create RDS instance
* Creast load balancer (http only)
* Create DSN records with Route53
    * connect to LB
* Create S3 buckets
    * Code
    * CDN
* Use CloudFront to distribute out the S3 bucket
* Create EC2 instance
    * Put it behind LB
    * install php, apache, etc
    * configure apache to redirect image access to CloudFront
    * create file for LB health check
    * install wordpress
* Backup code to **Code** bucket
    * `aws sync --recursive bla s3://bla`
    * add cron job to automate backup
* Copy WP pictues across to CDN bucket
    * `aws cp --recursive s3://code s3://cdn`
    * use apache URL rewrite to redirect picture access to CloudFront
    * setup cron job to sync local content to cdn
* Auto scaling
    * terminate the only instance
    * create launch configuration
    * add **user data** to provision code from code bucket
    * select subnet

## Configure Nginx 
     cd /etc/nginx/sites-available/
     sudo rm default 
     cd ../sites-enabled/
     sudo rm default 
     sudo ln -s ../sites-available/zoomapp .
     sudo /etc/init.d/nginx restart


# Links
* [complete guide](https://www.airpair.com/aws/posts/building-a-scalable-web-app-on-amazon-web-services-p1)
    * A Comprehensive Guide to Building a Scalable Web App on Amazon Web Services - Part 1 
* [quick start](https://aws.amazon.com/quickstart/)
    * Guide for deploying Magento
* [aws web app hosting](http://docs.aws.amazon.com/gettingstarted/latest/wah-linux/web-app-hosting-intro.html)
    * hosting Linux web app
    * nice basic graph of architecture
* [aws web app architecture](http://media.amazonwebservices.com/architecturecenter/AWS_ac_ra_web_01.pdf)
* [jayendra's blog](http://jayendrapatil.com/)
    * About AWS and especially cert
* [aws service limits](http://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html)

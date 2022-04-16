# Creating an Amazon ElastiCache Cluster

## Overview
- For an existing VPC, EC2 instance, create a ElasticCache Subnet Group, associate with (and create) a Memcached cluster (2 nodes, or type t2).
- Update the EC2 instance security group for INBOUND traffic on TCP, same port as set in the ElastiCache Memcache cluster, of the Subnet CIDR blocks. (This allows traffic from the cluster to the EC2 instance)
- In existing EC2 instance, we install ElasticCache Memcached Extension for PHP, since a PHP application, and the interpreter (AWS Memcached connector is able to automatically identify all of  the nodes in a cache cluster, and to initiate and maintain connections to all of these nodes, it connects to our configuration endpoint.)
- Then, using the endpoint point the php client at the endpoint. 

## Important note:
- Below notes are a combination of lab instructions, and my edits.
- Source: https://cloudacademy.com/
    - Credit where credit is due :)

## Create a cache Subnet Group

- An **ElastiCache Subnet Group** is a collection of subnets that can be assigned to any **Amazon ElastiCache cluster** running in an Amazon Virtual Private Cloud (VPC) environment. 
- ElastiCache uses that cache subnet group to select a subnet and IP addresses within that subnet to associate with the cache nodes.

### Steps to create Cache Subnet groups (multi-AZ)
- Go to AWS service ElastiCache
- From the ElastiCache dashboard, click on Subnet Groups link in the sidebar menu and then on Create Subnet Group blue button.
- Create a new Cache Subnet Group specifying the following basic information:
    - Name: ca-cache-subnets
    - Description: subnets for ca-cache
    - VPC ID: default VPC ID here
- Select two or more VPC Subnet IDs of any available Availability Zone, Add them and then click on Create:
- The Cache Subnet Groups now displays the "ca-cache-subnets" subnet group that you can use for deploying an ElastiCache cluster.

## Create a Memcached cluster using AWS Elasticache 

- Amazon ElastiCache enables you to set up, manage, and scale a distributed in-memory cache environment in the cloud. 
- It provides a high-performance, resizable, and cost-effective caching solution, while removing the complexity associated with deploying and managing a distributed cache environment.
- You can choose from Memcached or Redis protocol-compliant cache engine software, and let ElastiCache perform software upgrades and patch management for you automatically.

### Steps
- ElastiCache Dashboard
- For Cluster engine, select Memcached:
- Under Memcached settings, enter the following details:
    - Name: ca-cache
    - Engine Version: latest
    - Port: default
    - Parameter Group: default
    - Node Type - Select cache.t2.micro
    - Number of Nodes - Select 2
- In Advance Memchaed settings
    - select the subnet group created earlier **ca-cache-subnets**
- Create
- After created, take note of the **Configuration Endpoint**

## Configure Security Group (allow inbound traffic)

- The rules of a Security Group control the inbound traffic that's allowed to reach the instances that are associated with the security group and the outbound traffic that's allowed to leave them.
- By default, security groups allow all outbound traffic and disallow all inbound traffic.

### Steps
- in EC2, Security Groups
- select default
- updated security group TCP inbound rules as follows:
    - Note this relates to the subnet IPs of the ca-cache we created
    - i.e. must contain the CIDR blocks that the subnets are

## Connecting to Virtual Machine (using EC2 instance connect)
- Connect to existing EC2 instance

## Install ElasticCache Memcached Extension for PHP
- AWS Team developed an enhanced version of the Memcached connector adding the auto-discovery feature. 
- The AWS Memcached connector is able to automatically identify all of the nodes in a cache cluster, and to initiate and maintain connections to all of these nodes. 
- With Auto Discovery, the application does not need to manually connect to individual cache nodes; instead, it connects to a configuration endpoint. 
- The configuration endpoint DNS entry contains the CNAME entries for each of the cache node endpoints.
- In order to use the AWS Memcached connector for PHP, you need to execute the following instructions.

### Steps
- in EC2 instance, install the PHP interpreter
    - **sudo yum -y update**
    - **sudo yum install -y gcc make gcc-c++ php php-devel php-pear**
- Need to download and install the right version of the Memcached connector:
    - Check OS verison
        - uname -an
            - If the output contains the words "x86_64" you are using a 64bit AMI.
    - Check PHP version
        - php -v
            - The PHP version of the 64-bit Amazon AMI is 5.4.
- Download the connector
    - Using cloudAcademy's code here: 
        - **curl \
  --location https://github.com/cloudacademy/elasticache-cluster/raw/master/AmazonElastiCacheClusterClient-1.0.1-PHP54-64bit.tgz \
  --output AmazonElastiCacheClusterClient-1.0.1-PHP54-64bit.tgz**
    - Can find the latest client from the ElastiCache Console in the ElasticCache Cluster Client section
    - **sudo pecl install /home/ec2-user/AmazonElastiCacheClusterClient-1.0.1-PHP54-64bit.tgz**
- Enabling the Elasticache client:
    - **sudo /bin/sh -c 'echo "extension=amazon-elasticache-cluster-client.so" >> /etc/php.ini'**
- Ready to connect to the ElastiCache cluster now
    
## Connect to ElasticCache using PHP
- Let's test your Amazon ElastiCache cluster by connecting to it using a PHP client script.
- To download a demonstration PHP client script, enter the following:
    - **wget -O elasticache-client.php http://bit.ly/1vf3vQx** in EC2 instance
- Locate the Configuration Endpoint you made a note of earlier
- **sudo ln /usr/lib64/libsasl2.so.3 /usr/lib64/libsasl2.so.2**
- **php elasticache-client.php --endpoint=<YOUR-ELASTICACHE-ENDPOINT>**
    - in this example: **php elasticache-client.php --endpoint=ca-cache.grqpke.cfg.usw2.cache.amazonaws.com:11211**
## Destroy ElastiCache cluster
- in console, go to service, and delete. 

# Resources
- https://cloudacademy.com/ Lab

# lamejoke-cloudformation

WordPress blog extended from CloudFormation sample template provided by AWS:
**WordPress scalable and durable** from http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-applications-us-west-2.html

### Usage Instructions
This CloudFormation template will create a new WordPress site on EC2 with standalone MySQL database if no RDS snapshot is specified. If a RDS snapshot is listed, it will recreate the WordPress database using the snapshot and also load data from the specified S3 bucket.

This template assumes you already have a spare hosted zone avaliable in Route53 without an A record listed.

### Parameters

######BucketName
New parameter. This lists the S3 bucket you want to use to back up the WordPress directory & media files. If a RDS snapshot is specified, it will attempt to download bucket contents to the /var/www/html/ directory at stack creation. If not RDS snapshot is specified, it will attempt to create a new S3 bucket with the given parameter. If the S3 bucket already exists, stack creation will fail when CloudFormation fails to create a new bucket.

######DBAllocatedStorage
Same as in sample template. Specifies MySQL database size.

######DBClass
Same as in sample template. Specifies the database class. Default changed from db.t2.small to db.t2.micro to take advantage of free tier.

######DBName
Same as in sample template. Specifies the database name. You should use the same DBName if loading an RDS snapshot.

######DBPassword
Same as in sample template. Specifies the database password. You should use the same DBPassword if loading an RDS snapshot.

######DBSnapshotIdentifier
New parameter. Specifies the RDS snapshot to use if resuming an existing WordPress blog. RDS snapshots of the database are taken once a day and at stack deletion. If no snapshot is specified, this template assumes you are launching a fresh WordPress blog.

######DBUser
Same as in sample template. Specifies the database user. You should use the same DBUser if loading an RDS snapshot.

######EBSVolumeSize
New parameter. Defines the size of the EBS volume attached to EC2 instances.

######HostedZone
New parameter. This is the domain name of the Route53 hosted zone you will be using. The template will link its A-record to the Elastic Load Balancer created for this stack.

######InstanceType
Same as in sample template. Specifies the EC2 instance class. Default changed from t2.small to t2.micro to take advantage of free tier.

######KeyName
Same as in sample template. Without an EC2 KeyPair, you will not be able to SSH to the EC2 instance for troubleshooting.

######MultiAZDatabase
Same as in sample template. You can specify RDS to create the database in multiple availability zones for durability. (false for free tier)

######SSHLocation
Sam as in sample template. Default of 0.0.0.0/0 allows any IP in the world to SSH to the EC2 instance (using the EC2 KeyPair). You may improve security by locking down IP addresses allowed in. You can also edit the security group after the stack is launched.

######WebServerCapacity
Same as in sample template. WordPress is hosted on a EC2 instance in an AutoScaling group with size defined here. The default size of 1 means only one EC2 instance will run, and the AutoScaling group will launch a replacement EC2 instance if the existing one is terminated.

###Notes
WordPress stores posts and comments in its database, which means having multiple EC2 instances serving web traffic will not affect blog content. However media files and WordPress plugins are stored in the WordPress directory, which means EC2 web servers might not keep recent changes in sync. The cronjob (/etc/crontab) is configured to back up WordPress data every two hours.

The WordPress directory is also backed up using S3 instead of EBS snapshots, because EBS snapshots do not integrate well with AutoScaling Launch Configurations. (AutoScaling is intended for server groups where EC2 instances scale up and down based on demand)

The CloudFormation stack will launch in one of two states depending on the presence of the RDS snapshot: (a) creating a new WordPress blog or (b) loading and existing WordPress blog.
If the stack is in state (a), the launch configuration will launch new EC2 instances as new fresh WordPress web servers. Only in state (b) will it download WordPress data from S3 on new EC2 creation. Therefore it may be wise to launch a new CloudFormation stack with a RDS snapshot once a new site is created and WordPress data backed up to S3.

This template uses PHP version 5.6, instead of the deprecated PHP 5.3 in the original template.
Plugins used/supported so far: WordFence & WP Limit Logins

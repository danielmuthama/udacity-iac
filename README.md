## Deploy a high-availability web app using CloudFormation

### Project Overview

Suppose company is creating an Instagram clone like application. Developers pushed the latest version of their code in a zip file located in a public S3 Bucket.
Now the task is to deploy the application, along with the necessary supporting software into its matching infrastructure. This needs to be done in an automated fashion so that the infrastructure can be discarded as soon as the testing team finishes their tests and gathers their results.

### Infrastructure Diagram:

![udagram-infrastructure-diagram](https://user-images.githubusercontent.com/32739028/156870014-732cfd37-63b4-4978-9c16-80cd8d77d4ad.png)

### Project Files

* `/cloudformation-stacks`: 
  * `network-params.json`: Parameters file for network cloudformation stack.
  * `network.yml`: CloudFormation template for creating networking resources for this project.
  * `servers-params.json`: Parameters file for servers cloud formation stack.
  * `servers.yml`: CloudFormation template for creating servers for this project.
* `s3-bucket`: It contain the main html file of the application and upload status screenshot.
* `/scripts`: This folder contains script to create, delete and update CloudFormation stack
* `/workflow-screenshots`: This folder contain screenshots of whole workflow status
* `udagram-infrastructure-diagram.png`: This shows visual aid to understand the CloudFormation script.


### Project Setup

1. To create networking resources using cloudformation template, run the below command. After exceuting it these are the resources created:
  * VPC
  * Subnets
  * Route Tables
  * Routes
  * Internet Gateway
  * NAT Gateways

```
$ scripts/create.sh webapp-network-stack network.yml network-params.json
```

2. To create servers resources using cloudformation template which pulls source code from S3 bucket automatically while starting, run the below command. After exceuting it these are the resources created:
  * Security Groups
  * IAM Role, Policy, Instance Profile
  * Launch configuration
  * Auto Scaling Group
  * Load Balancer
  * Target Group

```
$ scripts/create.sh webapp-server-stack server.yml server-params.json
```





### Server specs
1. You'll need to create a Launch Configuration for your application servers in order to deploy four servers, two located in each of your private subnets. The launch configuration will be used by an auto-scaling group.
2. You'll need two vCPUs and at least 4GB of RAM. The Operating System to be used is Ubuntu 18. So, choose an Instance size and Machine Image (AMI) that best fits this spec.
3. Be sure to allocate at least 10GB of disk space so that you don't run into issues. 

### Security Groups and Roles
1. Since you will be downloading the application archive from an S3 Bucket, you'll need to create an IAM Role that allows your instances to use the S3 Service.
2. Udagram communicates on the default HTTP Port: 80, so your servers will need this inbound port open since you will use it with the Load Balancer and the Load Balancer Health Check. As for outbound, the servers will need unrestricted internet access to be able to download and update their software.
3. The load balancer should allow all public traffic (0.0.0.0/0) on port 80 inbound, which is the default HTTP port. Outbound, it will only be using port 80 to reach the internal servers.
4. The application needs to be deployed into private subnets with a Load Balancer located in a public subnet.
5. One of the output exports of the CloudFormation script should be the public URL of the LoadBalancer. Bonus points if you add http:// in front of the load balancer DNS Name in the output, for convenience.

### Other Considerations
1. You can deploy your servers with an SSH Key into Public subnets while you are creating the script. This helps with troubleshooting. Once done, move them to your private subnets and remove the SSH Key from your Launch Configuration.
2. It also helps to test directly, without the load balancer. Once you are confident that your server is behaving correctly, increase the instance count and add the load balancer to your script.
3. While your instances are in public subnets, you'll also need the SSH port open (port 22) for your access, in case you need to troubleshoot your instances.
4. Log information for UserData scripts is located in this file: cloud-init-output.log under the folder: /var/log.
5. You should be able to destroy the entire infrastructure and build it back up without any manual steps required, other than running the CloudFormation script.
6. The provided UserData script should help you install all the required dependencies. Bear in mind that this process takes several minutes to complete. Also, the application takes a few seconds to load. This information is crucial for the settings of your load balancer health check.
7. It's up to you to decide which values should be parameters and which you will hard-code in your script.
8. See the provided supporting code for help and more clues.
9. If you want to go the extra mile, set up a bastion host (jump box) to allow you to SSH into your private subnet servers. This bastion host would be on a Public Subnet with port 22 open only to your home IP address, and it would need to have the private key that you use to access the other servers.


### Solution
 Load balancer url : http://chall-Udagr-STQKNVDUIE98-720694535.us-east-1.elb.amazonaws.com
  
### How to run script with AWS CLI
Configure your aws IAM credentials in the commandline and run the following command;
```aws cloudformation create-stack --stack-name challengeStack --template-body file://udagram.yml  --parameters file://udagram.json --capabilities "CAPABILITY_IAM" "CAPABILITY_NAMED_IAM" --region=us-east-1```

> you can change the region to suit you.

# AWS Web App Refactoring

This project is called AWS Re-Architect Multi Tier Web Application Stack - VPROFILE. This will improve upon the Lift and Shift project by leveraging AWS managed services. 

## About The Project

It's a multi tier web application stack called Vprofile, it will be hosted and run in AWS using Refactoring Strategy.

## Objectives

- We want a flexible infrastructure
- No upfront costs
- Modernized effectively to improve performance
- IAC for automation
- Using managed services where applicable to boost agility
- Improve business continuity

## Vprofile Project Application Stack Diagram

![alt](/vprofile-project-aws-refactoring/project-diagram.png)


## Scenario

We have application services running on physical or virtual machines with various services that powers the application and the workload is in a local data center.

Managing all this on prem will need several teams; Virtualization team, Data Center OPs Team, Monitoring Team, Sys Admin, Security team, etc, involved.

 - **Problem**
  
   - Complex management
   - Scale UP/Down complexity
   - UpFront CapEx & Regular OPEx
   - Most of the processes will be manual
   - Difficult to automate and maintain
   - Time consuming operations
   
- **Solution**
  
  - Implementing a cloud solution that has the following advantages;
    - PayAsUgo
    - PAAS & SAAS services to have low operation overhead
    - IAAC
    - Flexibility/Elasticity (Scale out or  Scale in).
    - Easier to manage infrastructure
    - Automation
  
## AWS Services on this project

**Frontend Services**

- Beanstalk to replace EC2 Instances for Tomcat VM
- Beanstalk to replace Nginx LB
- Beanstalk for Autoscaling services
- S3/EFS Storage for shared storage
  
**Backend Services**

- RDS Instances for MySQL DB
- ELASTICACHE Service to replace MEMCACHED
- ACTIVE MQ to replace RabbitMQ
- Route 53 for private/public DNS service
- Cloudfront for Content Delivery Network (CDN)

**Services Comparison**

- Beanstalk vs Tomcat Ec2/VM
- ELB in Beanstalk vs NGINX LB/ELB
- Autoscaling vs NONE / Autoscaling
- EFS/S3 vs NFS / S3 / EFS
- RDS vs MySQL on VM/EC2
- ELASTICACHE vs MEMCACHED on VM/EC2
- ACTIVE MQ vs RabbitMQ on VM/EC2
- ROUTE 53 vs GODaddy / Local DNS
- Cloudfront vs NONE / Multi DCs Across the world

## The AWS Architecture Design for the Project

- EC2 Instances
- ELB from Beanstalk
- Autoscaling from Beanstalk
- EFS/S3 For Shared Storage
- RDS
- Elasticache
- ActiveMQ
- Cloudfront
- Amazon Certificate Manager
- Route 53

## User Experience Flow

- User(s) will access an HTTPS URL endpoint hosted by GoDaddy that will route the traffic to the load balancer endpoint. The traffic then will be routed to the Tomcat application server. 
- The HTTPS certificate will be issued by the Amazon Certificate Manager service.
- User access the HTTPS endpoint that will point to a load balancer in a Security Group that will only allow HTTPS traffic.
- Then the application load balancer will route the request to Tomcat instances using autoscaling group on HTTP port: 8080. It will only accept traffic configured in the load balancer's Security Group.
- Tomcat will allow all accesses to it through its own Security Group.
- Information of backend services i.e. **Memcache, RabbitMQ, MySQL** will be populated in **Amazon Route 53 DNS Private Zones** (replacing the **/etc/hosts** file on premises Linux OS). This will be accessed by the Tomcat services through private IPs.
- These backend services i.e. **Memcache, RabbitMQ, MySQL** will be in their own security group.

## Flow of Execution

- Log into [AWS Account](https://signin.aws.amazon.com/)
- Create Key Pairs
- Create Security Groups for Load Balancer, Tomcat and for Backend Services (Memcache, RabbitMQ, MySQL). Attach to an existing load balancer or new one.
- Launch Instances with user data (Bash Scripts) for provisioning
- Update Instances to name mapping in **Route 53**
- Build Application from source code on the local machine
- Upload the artifact to S3 bucket
- Download the artifact to the Tomcat EC2 Instance
- Set up ELB with HTTPS (Cert from Amazon Certificate Manager) *optional* if you don't have a domain to use
  **Note:** If you plan to use a new domain, you should know that DNS replication takes more than 48 hrs.
- Map ELB Endpoint to website name in GoDaddy DNS, *optional* if you don't have a domain to use
- Verify
- Build Autoscaling Group for Tomcat Instances

### Create a key pair

A key pair is used for ssh authentication when you want to access your resources remotely.

1. Go to **EC2 > Network & Security > Key Pairs** and hit **Create key pair**
2. Enter name **"vprofile-prod-key"** feel free to choose any name, but in keeping the standard that's why the key is given that name.
3. Under Private key file format select **.pem** for Linux or **.ppk** for Windows
4. Hit **Create key pair**
5. Change the key access permissions as per instructions indicated within the console.

### Security Groups Examples in this Project

1. Load Balancer Security Group - **vprofile-ELB-SG** 
   - Port: 443
   - Protocol: HTTPS - since it's public facing. If you don't have a domain yet, use HTTP.
2. Vprofile app Security Group - **vprofile-app-SG** 
   - Port: 8080 
   - Protocol: HTTP for Tomcat app and allow **vprofile-ELB-SG** so all traffic coming in is allowed to the web application.
   - Port: 22 
   - Protocol: SSH to allow traffic to Tomcat server for troubleshooting using a specific IP.

3. Vprofile Backend Services Security Group - **vprofile-backend-SG**
   - Port: 3306 add source as **vprofile-app-SG** to allow Tomcat traffic into MySQL DB
   - Port: 5672 add source as **vprofile-app-SG** to allow Tomcat traffic into RabbitMQ
   - Port: 11211 add source as **vprofile-app-SG** to allow Tomcat traffic into Memcache
   - Port: 22 to all backend services for troubleshooting using a specific IP.
   - All traffic on all ports with a source of **vprofile-backend-SG** to allow intercommunication among all services within the backend.

### EC2 VM instances setup

1. Clone the source code [here](https://github.com/hkhcoder/vprofile-project.git). If you just want the aws-LiftAndShift branch then download from [here](https://github.com/hkhcoder/vprofile-project/tree/aws-LiftAndShift)
2. **Create MySQL instance**
   - Tags: 
     - Key = "Name", Value = "vprofile-db01"
     - Key = "Project", Value = "vprofile"
   - Amazon Machine Image, search for **AlmaLinux**, select "AlmaLinux 8 x86_64" image
   -  Instance Type = t2.micro 
   -  Key pair = "vprofile-prod-key" as per above. 
   -  Network settings
      -  Select existing security group 
         -  "vprofile-backend-SG" 
   -  Configure Storage (6 GiB, gp2/3). 
   -  Go to *Advanced details* and paste the content of the script that you copy from the cloned repo inside **userdata** folder named, **"mysql.sh"** and paste it under "User data - *optional*" field.
   - Hit the *Launch instance* to create the instance.
3. **Create Memcache Instance**
   - Tags: 
     - Key = "Name", Value = "vprofile-mc01" 
     - Key = "Project", Value = "vprofile"
   - For Memcache, use same image, Key pair, and same security group, you only need to replace the script with the appropriate one .i.e **memcache.sh**
4. **Create RabbitMQ Instance**
   - Tags: 
     - Key = "Name", Value = "vprofile-rmq01" 
     - Key = "Project", Value = "vprofile"
     - For Memcache, use same image, Key pair, and same security group, you only need to replace the script with the appropriate one .i.e **rabbitmq.sh**
5. **Create Tomcat Instance**
   - Tags: 
     - Key = "Name", Value = "vprofile-app01"
     - Key = "Project", Value = "vprofile"
     - Amazon Machine Image
       - select "Ubuntu Server 22.04 LTS" image
     - Instance Type = t2.micro
     - Key pair = "vprofile-prod-key" as per above. 
     - Network settings 
       - Select existing security group
         - "vprofile-app-SG" 
     - Configure Storage
       - Value = 6 GiB
       - Value = gp2/3 
     - Go to *Advanced details* and paste the content of the script that you copy from the cloned repo inside **userdata** folder, named **"tomcat_ubuntu.sh"** and paste it under "User data - *optional*" field.
    
**Note**: Tomcat server is running on ubuntu, its security group is **"vprofile-app-SG"** and its service name is called **tomcat9**

To retrieve any instance user data in order to verify the scripts, you can paste this URL on your ssh terminal of any connected instance:
*curl* http://169.254.169.254/latest/user-data

### Backend services mapping using Route 53 (Private/Public DNS Setup)

#### Creating DNS public zone

To create a public hosted zone using the Route 53 console

1. Sign in to the AWS Management Console and open the Route 53 console at https://console.aws.amazon.com/route53/


2. If you're new to Route 53, choose Get started under DNS management.

   If you're already using Route 53, choose Hosted zones in the navigation pane.

3. Choose Create hosted zone.

4. In the Create Hosted Zone pane, enter the name of the domain that you want to route traffic for. You can also optionally enter a comment.

   For information about how to specify characters other than a-z, 0-9, and - (hyphen) and how to specify internationalized domain names, [see DNS domain name format.](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/DomainNameFormat.html)

5. For Type, accept the default value of Public Hosted Zone.

6. Choose Create.

7. Create records that specify how you want to route traffic for the domain and subdomains. For more information, see [Working with records.](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/rrsets-working-with.html)

8. To use records in the new hosted zone to route traffic for your domain, see the applicable topic:

   - If you're making Route 53 the DNS service for a domain that is registered with another domain registrar, see [Making Amazon Route 53 the DNS service for an existing domain.](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/MigratingDNS.html)

   - If the domain is registered with Route 53, see [Adding or changing name servers and glue records for a domain.](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/domain-name-servers-glue-records.html)

#### Creating DNS private zone

1. Go to *Route 53* > *Create hosted zone*
2. Domain name
   - "vprofile.com"
3. *Type*, select *Private hosted zone*
4. VPCs to associate with the hosted zone 
   - Region = US East(N.Virginia)
   - VPC ID select that appears. 
   **Note**: you need to select the region where you created your VMs. In my case, it's the above.
5. Hit the *Create hosted zone* button
6. Hit the *Create record* under the **vprofile** private zone.
   
   Record 1
      - Record name
        - "db01"
      - Record type
        - A
      - Value
        - Its private IP Address
      - TTL (seconds)
        - 300
      - Routing policy
        - Simple routing

   Record 1
      - Record name
        - "mc01"
      - Record type
        - A
      - Value
        - Its private IP Address
      - TTL (seconds)
        - 300
      - Routing policy
        - Simple routing

   Record 1
      - Record name
        - "rmq01"
      - Record type
        - A
      - Value
        - Its private IP Address
      - TTL (seconds)
        - 300
      - Routing policy
        - Simple routing

   

**Note**: Tomcat server will be able to resolve to the backend through the, **"application.properties"** file in the userdata folder. Because of that, there is no need to add it to the private DNS.

#### Joining your DNS public zone with your hosting

To join your DNS hosting service with AWS and create an SSL certificate using Amazon Certificate Manager (ACM), go [here](/vprofile-project-aws-LiftAndShift/Creating%20an%20SSL%20Certificate%20in%20AWS.md).

### Build Artifacts

1. Go to *src > main > resources > application.properties* file.
2. Update this file; "*jdbc.url=* **8jdbc:mysql://db01:3306/** *accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull*" to "*jdbc.url=* **jdbc:mysql://db01.vprofile.com:3306/** *accounts?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull*"
   - **Note**: Just make sure your URL doesn't have any gaps. 
3. For Memcache, it will be; *"memcached.active.host=mc01.**vprofile.com**"* 
5. For RabbitMQ, it will be; *"rabbitmq.address=rmq01.**vprofile.com**"*
6. Now to build the artifact, locate the folder you cloned from GitHub *cd* into it. You should see a pom.xml file.
7. Before you proceed you should have Apache Maven version 3 application installed and Java should be version 11
8. When ready execute;  *mvn install*, to build the artifact.
9.  If successful, you'll see newly created folder with a file called *"target/vprofile-v2.war"*.

### Deploy Artifacts

1. Go to *IAM >* scroll down and hit *Create access key > Command Line Interface (CLI)*
2. Check the box at the bottom to accept the risk and hit *Next > Create access key*

   **Note**: never expose your access keys to anyone.
3. Download the key and change the permissions of the key, recommended is chmod "400".
4. Go to your local terminal to log into AWS by running, *"aws configure"*
5. Enter your ID and Secret to continue
6. Run command; *"aws s3 mb s3://your-bucket-name"* to create one.
7. Copy the artifact to the bucket, run; "*aws s3 cp target/vprofile-v2.war s3://your-bucket-name/*"
8. Create a role and give it access permissions to the S3 and then attach it to the "vprofile-app01" instances to upload the file from S3 bucket. 
     - To create a role:
       - Go to *IAM > Roles > keep AWS Service selected*
       - *Use case = EC2* and hit *Next*, search for "AmazonS3FullAccess" policy and hit *Next*
       - *name* = "vprofile-EC2-access-s3" and hit *Create role*.
9.  Go to *EC2 > Instances > vprofile-app01 > Actions > Security > Modify IAM role >* in the drop down select your role and hit the *Update IAM role*.
10.  ssh into the Tomcat server
11.  Run 

         apt update && apt install awscli -y
12. To see if the IAM role that was attached to the EC2 has access permissions to the bucket.
    
         aws s3 ls
13. Run this command to copy the artifact 
    
         aws s3 cp s3://your-bucket-name/vprofile-v2.war /tmp/

## Setup and Verification on the EC2 Instance

1. Stop the Tomcat service 
   
         systemctl stop tomcat9
2. Remove the default install
   
         rm -rf /var/lib/tomcat9/webapps/ROOT
3. Copy the new artifact from the */tmp/* folder and rename it; 
   
         cp /tmp/vprofile-v2.war /var/lib/tomcat9/webapps/ROOT.war
4. Restart the service
   
         systemctl start tomcat9.service
         ls /var/lib/tomcat9/webapps/
5.  Verify the copied file
   
         cat /var/lib/tomcat9/webapps/ROOT/WEB-INF/classes/application.properties 
   
### Create Target Group and Load Balancer

1. Go to *EC2 > Target groups > Create target group > Instances*
   - name = vprofile-app-TG 
   - Protocol: HTTP
   - Port: 8080
   - Health checks 
   - Protocol: HTTP 
   - Health check path = /login 
   - Advanced health check settings 
     - Override and enter 8080
2. Hit *Next* and select the "vprofile-app-01" in the *Register Targets* and hit *Include as pending below*.
3. Hit *Create target group* to finish
4. Go to *Load Balancers* to create one and hit the *Create load balancer* button *> Application Load Balancer > Create*
   - Name = "vprofile-prod-ELB"
   - Network mapping = select all zones,
   - Security groups = "vprofile-ELB-SG" 
   - Default action = "vprofile-app-TG"
5. Click on *Add listener tag* 
   - Key = "Name"
   - Value = "HTTPS:443" 
   - Protocol = "HTTPS"
   - Port = 443 
   - Default action = "vprofile-app-TG"
- **Note**: Add this listener only if you have an SSL certificate installed.
1. *Secure listener settings* 
   - *Default SSL/TLS server certificate* = *From ACM*, *Certificate (from ACM)* = select from your domain.
2. Hit the *Create load balancer* button to finish 
3.  Copy your load balancer's DNS name to be updated in your DNS service provider, mine is GoDaddy. 
   - Go to *Managed DNS > DNS Records > Add New Record*. 
   - *Type = "CNAME", Name = "vprofileapp", Value = LB URL, TTL =* default value and hit the *Save* button.

### Create Autoscaling Group for Tomcat Instance

Auto scaling group require 3 things, namely; an AMI, a Launch Template and the Autoscaling group itself. Launch template provides a particular AMI, a Key pair and a Security group to be used by the Autoscaling Group.

#### AMI Image Creation

1. Go to *EC2 > Instances > vprofile-app01 > Actions > Image > Create Image*
2. *Create Image* 
   - Image name = "vprofile-app-image", 
   - Description = optional
3. Hit the *Create image* button.

#### Launch Template Creation

1. Go to *EC2 > Instances > Launch Templates*
2. Hit the *Create launch template* button
   - Name = "vprofile-app-LT"
   - Description = "vprofile-app-LT"
   - Application and OS Images > My AMIs > Owned by me
3. Instance type = "t2.micro", 
4. Key pair (login) = "vprofile-prod-key"
5. *Network Settings > Security Group > Select existing security group* = "vprofile-app-SG".
6. *Resource tags*
   - Tags: 
       - Key = "Name", Value = "vprofile-app". 
          Assign the tags to *Instances* and *Volumes*.
       - Key = "Project", Value = "vprofile". Assign the tags to *Instances*, *Volumes* and *Network interfaces*.
7. Advanced details
   - IAM instance profile = "vprofile-EC2-access-s3"
8. Hit the *Create launch template*

#### Auto Scaling group Creation

1. Go to *EC2 > Auto Scaling Groups > Create Auto Scaling group*
   - Name = "vprofile-app-ASG" 
   - Launch template = "vprofile-app-LT"
2. Hit the *Next*
3. Network 
   - VPC = select your VPC
   - Availability Zones and subnets = select all subnets and hit the *Next* button.
4. Load balancing
   - Attach to an existing load balancer
     - Choose from your load balancer target groups = "vprofile-app-tg | HTTP".
5. Health checks 
   - Check the *Turn on Elastic Load Balancing health checks* and hit the *Next* button to continue.
6. Configure group size and scaling policies - *optional*
    - Group size - *optional*
    - Desired capacity = 1 
    - Minimum capacity = 1 
    - Maximum capacity = 3
7. Scaling policies - optional 
   - Target tracking scaling policy 
   - Scaling policy name = leave the default value
   - Metric type = "Average CPU utilization"
   - Target value = 60
   - Instance warmup = 300.
8. Add notifications 
   - optional
     - Add notification.
9.  SNS Topic
      - use the billing alarm or create a new one and hit the Next button.
10. Add tags - optional 
    - Tags:
        - Key = "Name", "Value" = "vprofile-app"
        - Key = "Project", "Value" = "vprofile"
11. Hit *Next* to continue
12. Review and hit the *Create Auto Scaling group*

13. Terminate the "vprofile-app01" instance once you have an autoscaling group running.
14. Go to *Target Groups*, you should see your newly deployed instances.

### Validate 

1. Use your browser to open with your DNS URL or your load balancer URL.
2. Use username: admin_vp and password: admin_vp 
3. Go to RabbitMQ service to check queuing messages

## Sources & References

- [DevOps Beginners to Advanced with Projects](https://www.udemy.com/course/decodingdevops/?couponCode=LEADERSALE24A) by Imran Teli
- [Vprofile Source Code](https://github.com/hkhcoder/vprofile-project) by Imran Teli (hkhcoder)
- [DevOps Notes](https://visualpath.in/devopstutorials/devops) by Imran Teli
- [Architecture Diagramming](https://www.drawio.com/) courtesy of draw.io application
  
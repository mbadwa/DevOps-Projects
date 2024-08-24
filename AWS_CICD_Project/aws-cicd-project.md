# AWS CICD Project

Deploying an AWS CICD Project in AWS Cloud using AWS services. A manual CICD pipeline deployment. This is to take advantage of AWS services to streamline application delivery.

![AWS CICD Project](AWS%20CICD%20Project.png)


The Code will be migrated from GitHub to Bitbucket Repository so we can get exposure on how to manage code using Bitbucket.

## Outline

1. Create AWS Elastic Beanstalk
   - Build the code and deploy it on Beanstalk environment
2. Create an AWS RDS
   - RDS for Database connectivity
3. Bitbucket Repository
   - The source code will be stored in bitbucket
4. Migrate Source code from GitHub to Bitbucket
5. Build with AWS Code Build
   - Build code to create a .pom artifact
6. AWS Code Pipeline
   - AWS Pipeline will be used to connect all the services together instead on Jenkins for example.
7. Test CICD flow with git commit

### Bitbucket 

Bitbucket is a Git solution from Atlassian for professional teams .ie. Developers, DevOps, etc. There are many organizations that uses Bitbucket in real time environments. It uses a code toolchain, code and CICD powered by the Atlassian platform.

### Workflow

1. A dev will push code using git to GitHub since the code is already there. The code will be transferred to BitBucket hence forth.
2. The code changes will be managed by AWS CodeBuild will act like Jenkins Build, it will fetch the source code from Bitbucket Git Repository and build the source code into a .pom artifact.
3. AWS CodePipeline will deploy the artifact onto AWS Elastic Beanstalk service. CodePipeline like Jenkins will detect any new commit on the Bitbucket repository, fetch the source code, trigger the code build project which will build the source code into an artifact that'll be then deployed onto Elastic Beanstalk.
4. Amazon RDS will have schema tables for the application.

#### Steps

1. Setup Beanstalk Environment
   1. Create a key pair for Beanstalk Instance
        - Go to EC2 > Network & Security > Key Pairs > Create key pair
          - Key pair > Name
            - vpro-bean-key
        - Create key pair
   2. Create a Beanstalk Service Role
        - Go to IAM > Roles > Create role > Trusted entity type
          - Select AWS service
        - Use case > Service or use case
          - Select EC2
        - Hit Next
        - Permissions policies
          - Select:
            - AdministratorAccess-AWSElasticBeanstalk
            - AWSElasticBeanstalkCustomPlatformforEC2Role
            - AWSElasticBeanstalkWebTier
            - AWSElasticBeanstalkRoleSNS
          - Hit Next
          - Role name
            - vprofile-bean-role
          - Hit Create role
        
        Note: If you have a role named "aws-elasticbeanstalk-service-role" delete it. It was created before and it'll conflict with the one Beanstalk will try to create later.
   3. Create Beanstalk Environment
       - Search for Beanstalk > Create application
       - Select Web server environment
       - Application name
         - vprofile
       - Environment name
         - vprofile-prod
       - Domain
         - vprofile-prod-mbadwa
       - Platform > Platform
         - Tomcat
       - Platform branch
         - Tomcat 10 with Corretto 21 aka (JDK 21) on 64 bit Amazon Linux 2023
       - Platform version
         - Leave recommended
       - Scroll down to Presets > Configuration presets
         - Select Custom configuration
       - Hit Next
       - Service role
         - Create and use new service role
           - aws-elasticbeanstalk-service-role
         - EC2 key pair
           - vpro-bean-key
         - EC2 instance profile
           - vprofile-bean-role
       - Hit Next
       - Virtual Private Cloud (VPC) > VPC
         - Select Default or your preferred VPC
       - Instance settings
         - Public IP address > Check the Activated box
       - Instance subnets
         - Select all
       - Go all the way to Tags
         - Key: Project
         - Value: vprofile
       - Hit Next
       - Root volume (boot device)
         - Leave default
       - EC2 security groups
         - Leave blank, it'll create its own
       - Auto scaling group > Environment type
         - Select Load balanced
           - Instances:
             - Min: 2
             - Max: 4
         - Scroll down to Instance types
           - t2.micro(since it's Free tier)
         - Scaling triggers
           - Leave defaults
       - Load balancer network settings
         - Visibility
           - Select Public
       - Load Balancer Type
         - Application load balancer
         - Dedicated
       - Listeners
         - Leave defaults
       - Processes
         - Select "default"
         - Go to Actions > Edit
           - Go to Sessions > Session stickiness > Check the Enabled box
         - Hit Save
       - Leave everything to defaults
       - Hit Next
       - Rolling updates and deployments > Application deployments
         - Deployment policy
           - Select Rolling
         - Batch size type
           - Percentage
         - Deployment batch size
           - 50
        
            Note: In production you'll have multiple instances, 10% is ideal.
        - Leave everything else default and hit Next
        - Review and Edit where necessary
        - Hit Submit
   
2. Create RDS Database
   - Go to console and search for RDS > Create database
   - Choose a database creation method
     - Standard create
   - Engine options > MySQL
     - Edition
       - MySQL Community
     - Engine Version
       - MySQL 8.0.35 
   - Templates
     - Free tier
   - Availability and durability
     - Skip
    - Settings
      - DB instance identifier
        - vprords
      - Credentials Settings
        -  Master username
           - admin
      - Credentials management
        - Self managed
      - Check the Auto generate password box
   - Instance configuration
     - DB instance class
       - Burstable classes (includes t classes)
         - db.t3.micro
   - Storage
       - Leave defaults
   - Connectivity
     - Compute resource
       - Don’t connect to an EC2 compute resource
     - Virtual Private Cloud 
       - Select Default
     - DB subnet group
       - Default-vpc
     - Public access
       - No
   - VPC security group (firewall) > Create new
     - New VPC security group name
       - vprords-SG
     - Availability Zone
       - No preference
     - RDS Proxy
       - Skip
     - Certificate authority - optional
       - Leave default
   - Additional configuration
     - Database port
       - Leave default 
   - Tags - optional
     - Skip
   - Database authentication
     - Database authentication options
       - Password authentication
   - Monitoring
     - Skip
   - Additional configuration
     - Database options
       - Initial database name
         - accounts
       - DB parameter group
         - Leave default option
       - Option group
         - Leave default option
     - Backup
       - Enable automated backups
         - Leave it checked
       - Backup retention period
         - Leave default option
       - Backup window
         - Leave default option
       - Backup replication
         - Leave it unchecked
       - Encryption
         - Leave the Enable encryption option checked
       - Maintenance
         - Leave the Enable auto minor version upgrade option checked
       - Maintenance window
         - No preference
       - Deletion protection
         - Leave it unchecked
     - Estimated Monthly costs
       - FYI
     - Estimated monthly costs
       - FYI 
   - Hit Create database
   - Click on View credentials details
   - Save them somewhere safe

3. Allow RDS instance to communicate with EC2 Instances
   - Go to EC2 > Network & Security > Security Groups
   - Copy the RDS Security Group ID of a name similar to this:
  
            awseb-e-qj6rr4kyue-stack-AWSEBSecurityGroup-xjXJCNCSNUke

   - Copy its Security group ID
   - Go to vprords-SG > Inbound rules > Edit inbound rules
     - Add rule
       - Type: 
         - Custom Type
       - Protocol: 
         - TCP
       - Port range: 
         - 3306
       - Source: Custom
         - Value: The SG ID copied above
       - Description
         - Allow EC2 Instances Security Group
     - Hit Save rules
4. Test Connection from EC2 instances to the RDS Database
     - SSH in any of the vprofile-prod instances

            chmod 400 "vpro-bean-key.pem"
            ssh -i "vpro-bean-key.pem" ec2-user@107.20.56.101
            sudo -i

     - To test connection from the vprofile-prod instance to RDS DB we need MySQL agent installed. Select and copy  the dnf search result,"mariadb105".
  
            dnf search mysql
            dnf install mariadb105 -y
     - Go o Amazon RDS > Databases > vprords > Connectivity & security > Endpoint & port
       - Copy the endpoint
            
                # Command format (mysql, host (RDS Endpoint URL), username, and password)
                mysql -h paste-end-point-URL-here -u admin -p

        - Hit Enter
  
                Enter password:

        - When connected in DB prompt, type; "show databases;" command, you should see 'accounts' database listed. 

                MySQL [(none)]> show databases;
                +--------------------+
                | Database           |
                +--------------------+
                | accounts           |
                | information_schema |
                | mysql              |
                | performance_schema |
                | sys                |
                +--------------------+
                5 rows in set (0.001 sec)

            If not, create it.

                CREATE DATABASE accounts;
                exit

5. DB Initialization

   1. Open the [source code's](https://github.com/hkhcoder/vprofile-project) switch to [aws-ci](https://github.com/hkhcoder/vprofile-project/tree/aws-ci) branch. Go to [src](https://github.com/hkhcoder/vprofile-project/tree/aws-ci/src) > [main](https://github.com/hkhcoder/vprofile-project/tree/aws-ci/src/main) > [resources](https://github.com/hkhcoder/vprofile-project/tree/aws-ci/src/main/resources) > open [db_backup.sql](src/main/resources/db_backup.sql). Click on Raw and copy the [path](https://raw.githubusercontent.com/hkhcoder/vprofile-project/aws-ci/src/main/resources/db_backup.sql).
   2. Go to the instance
   
            wget https://raw.githubusercontent.com/hkhcoder/vprofile-project/aws-ci/src/main/resources/db_backup.sql

            mysql -h paste-end-point-URL-here -u admin -p accounts < db_backup.sql

        Hit Enter

            Enter password:
        
        Run "show tables;", you should see the table like below

            MySQL [(none)]> show tables;

            +--------------------+
            | Tables_in_accounts |
            +--------------------+
            | role               |
            | user               |
            | user_role          |
            +--------------------+
            3 rows in set (0.002 sec)

            MySQL [(none)]> exit;
6. Create Bitbucket Repository
   
   1. Create Account
      1. Go to [Bitbucket](https://bitbucket.org/product/) > Get it free > Create an account
      2. Hit Sign Up
      3. Confirm with code
      4. Enter Full name and Password
      5. Hit Continue
      6. Your new workspace
         - Enter a unique name
      7. Hit Agree and Create Workspace
   2. Create Repository
      1. Project name:
         - vprofile 
      2. Repository name:
         - vproapp
      3. Access level
         - Check Private repository box
      4. Include a README?
         - No 
      6. Include .gitignore?
         - No
      8. Hit Create repository

   3. Generate SSH keys on your local machine, name it "vpro_rsa"
      - Run the commands, the last one confirm the prompt with a "yes".

            ls ~/.ssh
            cd ~/.ssh
            ssh-keygen
            ls
            cat vpro_rsa.pub
            ssh -T git@bitbucket.org

   4. Copy the public key to Bitbucket account
       - Click on Settings gear > Personal Settings > Personal Bitbucket Settings > SSH Keys > Add key 
         - Label: name of your key
         - Key: Paste your key
         - Hit Add key
   5. Create SSH config file for bitbucket
        1. Create the file named config in the ~/.ssh folder

                sudo vim config

        2. Insert the content below, makes ure the indentation is the same. Third line going down have two spaces. Save and quit. Alternatively copy from [here](/config).

                # bitbucket.org
                Host bitbucket.org
                  HostName bitbucket.org
                  User git
                  PreferredAuthentications publickey
                  IdentityFile ~/.ssh/vpro_rsa

        3. Go to Bitbucket > Repositories > vproapp 
            - You'll see the clone command, make sure SSH is selected, copy the command
            - Clone the repo in temp just to test if SSH is working or not

                    cd /tmp/ && git clone git@bitbucket.org:mbadwa1/vproapp.git
                    
                    ls -ltr 
                    
                    # or 

                    cat vproapp/.git/config

                Note: You'll see the vproapp repo listed and the last command will display the path to the repo as well
7. Migrate GitHub Source Code Repo to Bitbucket
   1. Clone source code from GitHub
      1. Go to the source code in [GitHub](https://github.com/hkhcoder/vprofile-project/) > Expand Code by clicking on the arrow > Clone the code.
      2. Create a folder named AWS_CICD_Project or any name in your preferred location
      
              git clone https://github.com/hkhcoder/vprofile-project.git
              cd vprofile-project
              cat .git/config
              git brach -a
              git checkout aws-ci
              git fetch --tags

   2. Remove GitHub remote url
   
      - To remove URL and verify, run;
                
            git remote rm origin
            cat .git/config
                
   3. Add Bitbucket repository URL
      - To add add and verify, run:
      
            git remote add origin git@bitbucket.org:mbadwa1/vproapp.git
            cat .git/config
          
   4. Push the code
      - To push the code

            git push origin --all
            git push --tags

    Note: Replace the *git remote add origin bitbucket.org:mbadwa1/vproapp.git* with your repo details. The *git fetch --tags* is for fetching repos tags, in this case, there no tags in the repo, and the *git remote rm origin*, removes the GitHub URL from the repo
8. Code build with AWS CodeBuild
   
   Code Build is like Jenkins Build, the difference is that Jenkins is a server while CodBuild is a fully managed CI service that compiles source code, runs tests, and produces an artifact ready to deploy. It is Pay as You Go service. When you run the build, you get charged for the resources used during the build run time.

   1. Create a bucket for artifact storage
      - Make sure the bucket is in the same region as the CodeBuild
      - Go to Amazon S3 > Create bucket > Unique-Bucket-name
      - Hit Create bucket
   2. Create CodeBuild Project
      - Go to CodeBuild > Create project
        - Project configuration
          - Project name
            - vproappbuild
        - Additional configuration
          - Optional
        - Source > Source 1 - Primary > Source provider
          - Select Bitbucket
          - Hit Manage default source credential > Credential type
            - OAuth
          - Service
            - Secrets Manager (recommended)
          - Secrets
            - New secret
          - Hit Connect to Bitbucket
          - On the pop up click Grant access
            - Processing OAuth request
              - Secret name
                - name-secret
              - Secret description
                - Optional
            - Hit Confirm
            - You should see this confirmation; *Successfully connected through Secrets Manager secret*
            
            Note: Make sure you are logged into your Bitbucket using the same browser as your AWS Console. 
          - Repository
            - Repository in my Bitbucket account
          - Bitbucket repository
            - https://mbadwa-admin@bitbucket.org/mbadwa1/vproapp.git
          - Source version 
            - aws-ci
        - Primary source webhook events
          - Skip
        - Environment
          - Provisioning model
            - On-demand
          - Environment image
            - Managed image
          - Compute
            - EC2
          - Operating system
            - ubuntu
          - Runtime(s)
            - Standard
          - Image
            - aws/codebuild/standard:7.0
          - Image version
            - Leave default
          - Service role
            - New service role
          - Role name
            - Not sure yet
          - Additional configuration
          - Skip
        - Buildspec
          - Build specifications
            - Insert build commands
          - Build commands > Switch to editor
            - [Copy](https://github.com/hkhcoder/vprofile-project/blob/aws-ci/aws-files/build_buildspec.yml) and paste in your editor.
            - Replace the [application.properties](https://github.com/hkhcoder/vprofile-project/blob/aws-ci/src/main/resources/application.properties) file credentials with your RDS credentials

              1. RDS password 
  
                      # - sed -i 's/jdbc.password=password-in-application.properties-file/jdbc.password=saved-RDS-password/' src/main/resources/application.properties
                      - sed -i 's/jdbc.password=admin123/jdbc.password=nr1mTWY6OvlLBovvmZpD/' src/main/resources/application.properties
              2.  RDS username

                      # sed -i 's/jdbc.username=username-in-applications.properties/jdbc.username=current-RDS-username/' src/main/resources/application.properties 
                      sed -i 's/jdbc.username=admin/jdbc.username=admin/' src/main/resources/application.properties 
              3.  RDS Endpoint URL 

                      # sed -i 's/db01:3306/replace-with-RDS-endpoint-here/' src/main/resources/application.properties 
                      sed -i 's/db01:3306/vprodb.c50sgqqusvnr.us-east-1.rds.amazonaws.com:3306/' src/main/resources/application.properties
                  Note: Your buildspec file should look similar to this [one](build_buildspec.yml) and don't forget the forward slashes at the end of the password, Username and the URL.
              5.  Copy the [content](/build_buildspec.yml) and paste in Build commands in AWS. Fix any errors in the code.
        - Artifacts > Artifact 1 - Primary
          - Type
            - Amazon S3
          - Bucket name
            - your-bucket-name
          - Name
            - Skip
          - Path - optional
            - Skip
          - Namespace type - optional
            - None
          - Artifacts packaging
            - None
          - Disable artifact encryption
            - Leave it unchecked
          - Additional configuration
            - Skip
        - Logs > CloudWatch
          - Check the CloudWatch logs - optional box
          - Group name - optional
            - vprofileproject
          - Stream name prefix - optional
            - vprobuild
      - Hit Create build project
9. Build, Deploy & Code Pipeline
    1. Test the build
         - Go to Build projects > proappbuild > Hit on Start build
         - Open Phase details tab
         - Here you'll see the build progress
    2. Deploy to S3 bucket
         - Go to CodePipeline > Getting started > Create pipeline
         - Pipeline settings > Pipeline name
           - vprodcicdpipeline
         - Scroll down leave everything default
         - Hit Next
         - Source provider
           - Select Bitbucket
         - Connection
           - Hit Connect to BitBucket
         - On the pop up Create a connection > Create Bitbucket connection > Connection name
           - bitbucketvprofile
         - Hit Connect to Bitbucket
         - Grant access
         - Bitbucket apps
           - Install a new app
         - Hit Grant access
         - Hit Connect 
         - You should see this message: *Ready to connect
Your Bitbucket connection is ready for use.*
           - Repository name
             - Select your-workspace/vproapp
           - Default branch
             - aws-ci
           - Output artifact format
             - CodePipeline default
         - Trigger 
           - Trigger type
             - Trigger type
         - Hit Next
         - Build - optional > Build provider
           - AWS CodeBuild
         - Region
           - Leave default
         - Input artifacts
            - SourceArtifact
         - Project name
           - Select your project name
         - Build type
           - Single build
         - Hit Next
         - Deploy - optional
           - AWS ElasticBeanstalk
         - Region
           - Leave selected region
         - Input artifacts
           - BuildArtifact
         - Application name
           - vprofile
         - Environment name
           - vprofile-prod
         - Hit Next and Review
         - Hit Create pipeline
         - Go to Elastic Beanstalk > Environments > vprofile-prod > Events to check progress
         - Also in Application: vprofile > Application versions
           - You'll see a new app deployed.
         - Go to Elastic Beanstalk > Environments > vprofile-prod > Click on Domain
         - Log in with these credentials

                username: admin_vp 
                password: admin_vp
10.  Change the code trigger a new build
         
      - Go to your local repo and edit README.md and add extra # in 2 or remove the #. Save and quit
    
            ls 
            sudo vim README.md
            git add .
            git commit -m "test aws pipeline trigger"
            git push origin aws-ci              
11. Clean Up
    1.  Delete RDS 
          - Go to RDS > Databases > vprords > Actions > Delete
          - Uncheck all boxes and check the acknowledgement box
          - Hit Delete
    2.  Delete Beanstalk
          - Go to EC2 > Network & Security > Security Groups > vprords-SG
          - Go to Inbound rules > Edit inbound rules 
            - Delete the rule with a security group.
          - Go to AWS Console Home > Elastic Beanstalk > Applications > Select vprofile-prod > Actions > Delete 
          - Enter name
            - vprofile
          - Hit Delete


# References

1. [Buildspec File](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)
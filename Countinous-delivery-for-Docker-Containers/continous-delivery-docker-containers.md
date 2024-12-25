# Continuous Delivery For Docker Containers

This project will be continuously building Docker containers and continuously deploying containers to Kubernetes cluster

## Scenario

- Currently there is a Microservices Architecture of an Application in place
- The Application is Containerized
- There is Continuous Code Changes
- There is Continuos Build & Test
- Regular Build of Container Images
- Regular Deployments Requests to Ops Team

## Problem

- Operations team in charge of Managing containers Gets continuous Deployment Requests
- Manual Deployment creates dependency
- Time Consuming

## Solution

- Automate Build & Release process
- Build Docker Images & Deploy Continuously as fast as the Code commits
- Create a Continuous Deployment Process

### Project setup requirements
  
- **Tools**

  - Project Source code - [GitHub Repository](https://github.com/devopshydclub/vprofile-project/tree/cicd-kube)
  - Domain - GoDaddy
  - Infra Deploy Environment - AWS Cloud
  - Container Orchestration Tool - Kubernetes 
  - Docker Container Run time and Build Tool - Docker Engine
  - CICD server - Jenkins
  - Container Registry - DockerHub
  - Packaging and Deploying Docker Images, complete Application Stack on Kubernetes - Helm
  - Version Control System - Git
  - Application Build Tool - Maven
  - Code Analysis - SonarQube Server
   
## Objective

- Achieve Continuous Delivery for Docker Containers


## Architecture Continuous Delivery Pipeline

![alt](./img/Docker-CICD%20for%20Docker%20K8s%20on%20Jenkins.drawio.png)


### Flow Of Execution

1. Continuous Integration Setup

   a. Jenkins
   
   b. SonarQube
   
   c. Nexus (Continuous Integration Project)

2. Dockerhub account & Jenkins Setup
   1. Store Dockerhub Credentials in Jenkins
   2. Setup Docker Engine in Jenkins
   3. Install Plugins in Jenkins
   
      a. Docker-pipeline
   
      b. Docker

      c. Pipeline utility
3. Create Kubernetes Cluster with Kops
4. Install Helm in Kops VM & Create Helm Charts
5. Test Charts in K8s Cluster in a test namespace
6.  Add Kops VM as Jenkins Slave
7.  Create Pipeline code (Declarative)
8.  Update Git Repository with

    a. Helm Charts
    
    b. Dockerfile
    
    c. Jenkinsfile (Pipeline code)
9.  Create Jenkins job for Pipeline
10. Run & Test the job

### 1. Continuos Integration Setup

1. Set up servers
   
   - The Jenkins server setup instructions can be found [here](https://github.com/mbadwa/devops-notes/blob/main/Jenkins-notes/jenkins-deep-dive.md#1-install-jenkins-in-aws)
   - The SonarQube server setup instructions can be found [here](https://github.com/mbadwa/devops-notes/blob/main/Jenkins-notes/jenkins-deep-dive.md#3-server-resources-setup)
   - The Nexus server setup instructions can be found [here](https://github.com/mbadwa/devops-notes/blob/main/Jenkins-notes/jenkins-deep-dive.md#3-server-resources-setup)

   NOTE: Nexus is optional since we will use DockerHub as the Repository

2. Integrating Jenkins server with SonarQube server
   
     1. Log into your SonarQube instance to create a key
     
         - Go to the Administrator icon > My Account > Security
           - Generate Tokens >  Name: `kube-jenkins`, Type: `User Token`, Expires in: `30 days`
           - Hit Generate
           - Copy it     
     
     2. Go to Jenkins > Manage Jenkins > Plugins > Available plugins > search for SonarQube
        - Check the `Sonar Quality Gates` and `SonarQube Scanner`
        - Hit Install
     
     3. Go to Jenkins > Manage Jenkins > System > SonarQube servers > Add SonarQube

         - Name: `sonar-pro`
         - Server URL: http://SonarQube-pvt-IP
         - Hit the Save button
     
     4. Go back to Go to Jenkins > Manage Jenkins > System > SonarQube servers > Add SonarQube
         
         - Server authentication token > Add > Select `Jenkins`
         - Domain: Global credentials (unrestricted)
         - Kind: Secret text
           - Scope: Global(Jenkins, nodes, items, all child items, etc)
           - Secret: Paste your SonarQube token
           - ID: `kube-sonar-token`
           - Description: `kube-sonar-token`
         - Hit Add
         - Select the `kube-sonar-token`
         - Hit Save
   
     5. Allow traffic from SonarQube to Jenkins instance
     
        Go to EC2 > Security Groups >  Jenkins-SG > Inbound rules > Edit inbound rules > Add rules

        - All traffic, Source: `SonarSG`
        - Save rules

     6. Allow traffic from Jenkins to SonarQube instance
   
        Go to EC2 > Security Groups >  SonarSG > Inbound rules > Edit inbound rules > Add rules
        - All traffic, Source: `Jenkins-SG`
        - Save rules

### 2. Dockerhub account & Jenkins Setup
   
   1. Log into [Dockerhub](https://hub.docker.com/)
   2. Create DockerHub credentials in Jenkins

      - Go to Jenkins > Manage Jenkins > Credentials > Stores scoped to Jenkins > System
        - Global credentials (unrestricted) > Add credentials
          - Kind: Username with password
            - Scope: Global(Jenkins, nodes, items, all child items, etc)
            - Username: your-Dockerhub-username
            - Password: your-DockerHub-password
            - ID: `dockerhub`
            - Description: `dockerhub`
          - Hit Create

   3. Install Docker Engine in the Jenkins server
   
      - SSH into the Jenkins server
      
            $ ssh -i "jenkins-key.pem" ubuntu@your-server-pub-IP

      - Installation instructions [here](https://docs.docker.com/engine/install/ubuntu/)
      - Check is successfully installed
      
            $ systemctl status docker
      
      - Update password and add Jenkins user to the Docker group so jenkins user can also run Docker
      
            $ sudo passwd jenkins
            $ sudo usermod -aG docker jenkins
            $ id jenkins
            $ su - jenkins
            $ docker images
            $ exit
            $ sudo reboot
   
      NOTE: The reason Docker Engine is installed in Jenkins server is due to the fact that we will run build commands within the server

   4. Install Plugins in Jenkins
   
      1. Go to Manage Jenkins > Plugins > Available plugins 

         - Search for `Pipeline Utility Steps`, `Docker` and `Docker Pipeline`
         - Hit Install
   
   5. Install Maven in Jenkins Server
      
      1. Download the [Maven binaries](https://maven.apache.org/download.cgi), right-click on the `Binary tar.gz archive` and click on `Copy link address`
   
               $ mvn -version
               $ wget https://dlcdn.apache.org/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.tar.gz
               $ tar -xvf apache-maven-3.9.9-bin.tar.gz
               $ sudo mv apache-maven-3.9.9 /opt/
      
      2. Setting Maven Home and Path
      
      
               $ MVN_HOME='/opt/apache-maven-3.9.9'
               $ PATH="$MVN_HOME/bin:$PATH"
               $ export PATH
               $ source .profile
      
      3. Verify version
      
               $ mvn -version
            
### 3. Create Kubernetes Cluster with Kops

To set up Kops, you'll require to have a Domain that you can set up in AWS. 
Without this you won't be able to install Kops. 
[Here](https://github.com/mbadwa/devops-notes/blob/main/Kubernetes-notes/kubernetes-notes.md#setup-with-kops)
are the instructions of Kops setup.

Note: After installing KOps, it'll be running in autoscaling mode with one Control node and two nodes. 
You'll need to stop the cluster by following the instructions [here](https://github.com/mbadwa/devops-notes/blob/main/Kubernetes-notes/kubernetes-notes.md#setup-with-kops) to save on costs.
You can turn it back on when done with rest of the setup.

### 4. Install Helm in KOps Instance & Setup Git Repo Helm Charts

   1. To install helm, go to Quick start [Install Helm](https://helm.sh/docs/intro/quickstart/). Installation guide [here](https://helm.sh/docs/intro/install/) and you can download binary version [here](https://github.com/helm/helm/releases)

      1.  Copy the binary version URL and download
      
               $ cd /tmp
               $ wget https://get.helm.sh/helm-v3.16.2-linux-amd64.tar.gz
      
      
      2. Extract the binary file

               $ tar -zxvf helm-v3.16.2-linux-amd64.tar.gz
      
      3. move the binary file to Helm folder 
   
               $ cd linux-amd64/
               $ sudo mv helm /usr/local/bin/helm
               $ helm version
   
   2. Helm Charts & git repo Setup

      1. Log into your KOps instance
      
             $ ssh -i "kops-key.pem" ubuntu@your-kops-public-IP
      
      2. Create a repo in [GitHub](https://github.com/mbadwa/cicd-kube-docker)
      
      
      3. Clone your repo in the Kops server using the https URL
      
             $ git clone your-GitHub-https-repo-URL
      
      4. Clone the source code repo to fetch data from it
      
             $ git clone https://github.com/devopshydclub/vprofile-project.git
      
      5. Switch to `vprofile-project` folder & to `vp-docker` git branch
      
             $ cd vprofile-project
             $ git checkout vp-docker
             $ ls
             $ git status
      
      6. Copy the contents of `vp-docker` to `cicd-kube-docker` repo
      
             $ cp -r * ../cicd-kube-docker
             $ cd ../cicd-kube-docker
             $ ls
      
      7.  Remove unnecessary folders and files
      
              $ rm -rf Docker-db Docker-web compose ansible helm
              $ ls 
      
      8.  Move the Dockerfile from `Docker-app` folder to the current folder
      
              $ mv Docker-app/Dockerfile .
              $ rm -rf Docker-app
      
      9.  Folder and files remaining after clean up
      
              $ ls

          Output

              Dockerfile  README.md  kubernetes  pom.xml  src
      
      10. Create a helm directory and a helm chart
      
              $ mkdir helm && cd helm
              $ helm create vprofilecharts && ls
              $ cd vprofilecharts && ls
              $ cd templates
              $ rm -rf *
      11. Copy the kubernetes definitions files from `kubernetes` folder to `/helm/vprofilecharts/templates/` folder
      
              $ cd ../../../
              $ cp kubernetes/vpro-app/* helm/vprofilecharts/templates/
              $ ls helm/vprofilecharts/templates/
              $ cd helm/vprofilecharts/templates/
      
      12. Change the image versioning from static to dynamic iterations using variables
   
            Remove the  `vprofile/vprofileapp`  to replace with `{{ .Values.appimage}}`

              $ vim vproappdep.yml

            Note: Variables can be passed using `values.yaml` file in `/helm/vprofilecharts/` folder, just like a `.tfvars ` file in Terraform
      
      13. Test the Helm charts in `helm/vprofilecharts/templates/` folder
      
          - Create a test namespace and install the helm in it, make sure you are in `/cicd-kube-docker` folder
              
                $ kubectl create namespace test
                $ helm install --namespace test vprofile-stack helm/vprofilecharts --set appimage=imranvisualpath/vproappdock:9

            Output
         
                  NAME: vprofile-stack
                  LAST DEPLOYED: Sat Nov 16 20:49:45 2024
                  NAMESPACE: test
                  STATUS: deployed
                  REVISION: 1
                  TEST SUITE: None
                  
          - Check the kubernetes deployment
          
                $ helm list --namespace test
                $ kubectl get all --namespace test
          
          - Delete the test deployment
          
                $ helm delete vprofile-stack --namespace test
          
          - Create a prod namespace

                $ kubectl create namespace prod

          - Push the changes to GitHub repo
  
                $ git add .
                $ git commit -m "helm charts"
                $ git push origin main
         
         NOTE: If you stopped your Kubernetes Kops cluster, you need to restart it 
         by following the instructions 
         [here](https://github.com/mbadwa/devops-notes/blob/main/Kubernetes-notes/kubernetes-notes.md#setup-with-kops).

         You may encounter this error when restarting the cluster, 
         this is due to Kubernetes not recognizing the credentials

            error: You must be logged in to the server (the server has asked for the client to provide credentials)

         Solution

            kops export kubeconfig --admin --state=s3://kops-mbadwa-bucket
   
   3.  Add a rule in Kops server to allow Jenkins SSH traffic
   
      Go to EC2 > Security > kops-SG > Edit inbound rules > Add rule

      - Type: SSH, Custom: Jenkins-SG, Description - optional: Allow SSH from Jenkins
      - Hit Save rules

   4. Add Kops to Jenkins as a node
   
      1. SSH into Kops server

             $ ssh -i "kops-key.pem" ubuntu@ec2-pub-IP
             $ sudo mkdir /opt/jenkins-node
             $ sudo apt install openjdk-17-jdk -y
             $ java -version
   
      2. Install Maven
             $ sudo install 
      3. Create SSH user

         1. Create a user & change the `/opt/jenkins-node` folder to `jenkops`
         
                  $ sudo adduser jenkops
                  $ ls -l /opt/jenkins-node/
                  $ sudo chown jenkops.jenkops /opt/jenkins-node -R

         2. Switch editors to vim or keep nano (your preference) and grant `jenkops` admin rights

                  $ sudo update-alternatives --config editor
                  $ sudo su
                  # visudo 
        
         3. Add `jenkops` to admin users
         
                  # User privilege specification
                  jenkops ALL=(ALL:ALL) ALL
         
         4. Switch to `jenkops`
         
                  $ sudo su jenkops
                  $ cd /home/jenkops
         
         5. Create .ssh folder in `jenkops` user
         
                  $ mkdir ~/.ssh; cd ~/.ssh/ && ssh-keygen -t rsa -m PEM -C "Jenkops agent key" -f "jenkopsAgent_rsa"
         
         6. Add the public SSH key to the list of authorized keys on the agent machine
         
                  $ cat jenkopsAgent_rsa.pub >> ~/.ssh/authorized_keys
         
         7. Ensure that the permissions of the ~/.ssh directory is secure, 
            as most ssh daemons will refuse to use keys that have file permissions that are considered insecure:
         
                  $ chmod 700 ~/.ssh
                  $ chmod 600 ~/.ssh/authorized_keys ~/.ssh/jenkopsAgent_rsa
         
         8. Copy the private SSH key (~/.ssh/jenkopsAgent_rsa) from the agent machine to your OS clipboard (eg: xclip, pbcopy, or ctrl-c).
        
                  $ cat ~/.ssh/jenkopsAgent_rsa

            The output should be similar to this:
               
                  -----BEGIN RSA PRIVATE KEY-----
                  ...
                  -----END RSA PRIVATE KEY-----

      4. Go to Dashboard > Manage Jenkins > Nodes > New node 
      
            - Node name: KOps
            - Type: Permanent Agent
            - Hit Create
            - Description: KOps Jenkins node
            - Number of executors: 1
            - Remote root directory: `/opt/jenkins-node`
            - Labels?: KOPS
            - Usage: Only build jobs with label ....
            - Launch method: Launch agents via SSH
              - Host: `private-IP-of-KOps-EC2-instance`
              - Credentials: click on Add > Jenkins 
                - Kind: SSH Username with private key
                  - Scope: Global(Jenkins,nodes,items,all child items,etc)
                  - ID: kops-login
                  - Description: kops-login
                  - Username: jenkops
                  - Private Key
                    - Enter directly, paste the KOps instance private key (`cat ~/Downloads/kops-key.pem`)
                    - Add
                  - Select the `kops-login` key
                - Add
              - Host Key Verification Strategy: Non verifying Verification Strategy
            - Availability: Keep this agent online as much as possible
            - Hit Save
   
   5. Create a Jenkinsfile for pipeline deployment
   
      1. Clone the `ci-cd-kube-docker` repo to computer
      
             $ git clone https://github.com/mbadwa/cicd-kube-docker.git
      
      2. Create a Jenkinsfile in the folder and paste the code from the `Jenkinsfile` [here](https://github.com/devopshydclub/vprofile-project/tree/paac)
      
         Use any code editor and create a file named `Jenkinsfile`
      
      3. Edit the sections to align with the helm charts
      
         - Remove the Nexus stage `stage("Publish to Nexus Repository Manager")`
         - Under the `environment ` section, where variables are define, 
           remove all NEXUS variables and add two more variables, namely; `registry = "your-docker-hub-user/vproapp"`
         - Under the `environment` section, add `registryCredential=your-jenkins-token` located
           under Dashboard > Manage Jenkins >  Credentials > Stores scoped to Jenkins > System > Global credentials (unrestricted), copy the name; `dockerhub`
      
      4. Add a Sonar Scanner Tool in Jenkins

         Go to Jenkins Dashboard > Manage Jenkins > Tools > SonarQube Scanner installations > SonarQube Scanner

         - Name: `mysonarscanner4`
         Hit Save
      
      5. Test the pipeline

         Go to Jenkins Dashboard > New item > Enter an item name

         - `kube-cicd`
         - Select `Pipeline`
         - Hit OK
         - Scroll to Pipeline > Definition 
           - Select Pipeline Script from SCM
           - SCM: Git
           - Repositories > Repository URL
             - Paste the URL
           - Hit Save
         - Hit the Build to test
        
        Result  

            Checking status of SonarQube task 'AZPGMAeP62fy3_XlIszS' on server 'sonar-pro'
            SonarQube task 'AZPGMAeP62fy3_XlIszS' status is 'IN_PROGRESS'
            Cancelling nested steps due to timeout

      6. Webhook setup

         - Go to Projects > Select `vprofile-repo` > Project Settings > Webhooks
           - Hit Create
             - Name: `jenkins-webhook`
             - URL: `http://jenkins-server-pvt-IP:8080/sonarqube-webhook`
      
      7. Go to Jenkins to run the Build again


# References

1. [Jenkins versions error](https://stackoverflow.com/questions/72227442/unsupported-class-file-major-version-61)
2. [Maven Packages](https://maven.apache.org/download.cgi)
3. [Maven Installation](https://phoenixnap.com/kb/install-maven-on-ubuntu)
   
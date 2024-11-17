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

Th

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
           - Generate Tokens: `kube-jenkins`
           - Hit Generate
           - Copy it     
     
     2. Go to Jenkins > Manage Jenkins > System > SonarQube servers > Add SonarQube

         - Name: `sonar-pro`
         - Server URL: http://SonarQube-pvt-IP
         - Hit the Save button
     
     3. Go back to Go to Jenkins > Manage Jenkins > System > SonarQube servers > Add SonarQube
         
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
   
     4. Allow traffic from SonarQube to Jenkins instance
     
        Go to EC2 > Security Groups >  Jenkins-SG > Inbound rules > Edit inbound rules > Add rules

        - All traffic, Source: `SonarSG`
        - Save rules

     5. Allow traffic from Jenkins to SonarQube instance
   
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
      
      - Add Jenkins user to the Docker group so jenkins user can also run Docker
      
            $ sudo usermod -aG docker jenkins
            $ id jenkins
            $ su - jenkins
            $ docker images
            $ reboot
   
      NOTE: The reason Docker Engine is installed in Jenkins server is due to the fact that we will run build commands within the server

   4. Install Plugins in Jenkins
   
      1. Go to Manage Jenkins > Plugins > Available plugins 

         - Search for `Pipeline Utility Steps`, `Docker` and `Docker Pipeline`
         - Hit Install
   
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

          Dockerfile README.md kubernetes pom.xml src
      
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

   4. Add Kops to Jenkins as a slave
   
      1. SSH into Jenkins server

            $ ssh -i "kops-key.pem" ubuntu@ec2-pub-IP
            $ sudo mkdir /opt/jenkins-slave
            $ sudo apt install openjdk-8-jdk -y
            $ sudo chown ubuntu.ubuntu /opt/jenkins-slave -R
            $ java -version
   
      2. Go to Dashboard > Manage Jenkins > Nodes > New node 
      
            - Node name: KOps
            - Type: Permanent Agent
            - Hit Create
            - Description: KOps slave node
            - Number of executors: 1
            - Remote root directory: `/opt/jenkins-slave`
            - Labels?: KOPS
            - Usage: Only build jobs with label ....
            - Launch method: Launch agents via SSH
              - Host: `private-IP-of-KOps-EC2-instance`
              - Credentials: click on Add > Jenkins 
                - Kind: SSH Username with private key
                  - Scope: Global(Jenkins,nodes,items,all child items,etc)
                  - ID: kops-login
                  - Description: kops-login
                  - Username: ubuntu
                  - Private Key
                    - Enter directly, paste the KOps instance private key (`cat ~/Downloads/kops-key.pem`)
                    - Add
                  - Select the `kops-login` key
                - Add
              - Host Key Verification Strategy: Non verifying Verification Strategy
            - Availability: Keep this agent online as much as possible
            - Hit Save

      3. KOps
   
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
         - 
      
      4. jkk
   
   6. hjfdhf

   
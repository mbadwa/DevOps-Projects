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

2. Dockerhub account (Containerization Project)
3. Store Dockerhub Credentials in Jenkins
4. Setup Docker Engine in Jenkins
5. Install Plugins in Jenkins
   
   a. Docker-pipeline
   
   b. Docker

   c. Pipeline utility
6. Create Kubernetes Cluster with Kops
7. Install Helm in Kops VM
8. Create Helm Charts
9. Test Charts in K8s Cluster in a test namespace
10. Add Kops VM as Jenkins Slave
11. Create Pipeline code (Declarative)
12. Update Git Repository with

    a. Helm Charts
    b. Dockerfile
    c. Jenkinsfile (Pipeline code)
13. Create Jenkins job for Pipeline
14. Run & Test the job

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

### 3. Dockerhub account
   
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

### 4. Install Plugins in Jenkins
   
   1. Go to Manage Jenkins > Plugins > Available plugins 

      - Search for `Pipeline Utility Steps`, `Docker` and `Docker Pipeline`
      - Hit Install
   
### 5. Create Kubernetes Cluster with Kops

To set up Kops, you'll require to have a Domain that you can set up in AWS. Without this you won't be able to install Kops. [Here](https://github.com/mbadwa/devops-notes/blob/main/Kubernetes-notes/kubernetes-notes.md#setup-with-kops) are the instructions of Kops setup.

### 6. Install Helm in KOps Instance

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
   
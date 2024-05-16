# Vprofile project on prem

## Intro

There are a number of services/components that powers the vprofile Java web app. This entails having a run book to set up the project stack. This is an attempt to document the whole project. The project is to set up a web application.

## Objectives

- Learn VM automation locally
- Act as a baseline for upcoming projects
- Real world scenario setup on prem/local machine for R&D.

## Vprofile Project Application Stack Diagram

![alt](project-diagram.png)
  
## Project setup

- [Scenario]()
  - Problem 
    - You have a requirement to make changes to prod environment but you lack confidence to make those changes live and you want to create your own sandbox.
    - Local setup is complex.
    - Time consuming and not repeatable.
  - Solution
    - Have an automated local setup.
    - Use IAAC so it's repeatable.
  
- [Tools]()
  - Local env setup
    - Hypervisor - Oracle VM VirtualBox
    - Automation - Vagrant
    - CLI - bash in Linux
    - IDE - Visual Studio Code or your preference
 
- [The Architecture of Project Services]()
  - NGINX
  - TOMCAT
  - RABBITMQ
  - MEMCACHED
  - MYSQL
- [The Architecture of the Automated Setup]()
  - Vagrant
  - VirtualBox
  - Bash
  - Scripts

## User Experience Flow

- User opens a browser and enter the IP of a load balancer that routes the request to Tomcat server.
- Apache Tomcat is a Java web application service that hosts apps written in Java.
- If there is need for a shared storage or centralized storage the we will use NFS.
- The user login details will be stored in MySQL database service. A user attempts to login, the Tomcat app will run an SQL query to access user info stored in MySQL database server passing through a Memcache Database service.
- The user query will be sent back to Tomcat and then cached in Memcached so that the use gets validated using the caching service instead.
- RabbitMQ is a message broker/queuing agent service that streams data between to services(it's a dummy service in our env).

## Flow of Execution

1. Set up tools mentioned above
2. Clone source code [here](https://github.com/hkhcoder/vprofile-project.git)
3. cd into the vagrant directory
4. Bring up the VMs
5. Validate all VMs
6. Set up All the services
     - MySQL
     - Memcached
     - Rabbit MQ
     - Tomcat
     - Nginx
     - App Build and Deploy
7. Verify from browser
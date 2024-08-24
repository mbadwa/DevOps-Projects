# Containerization Of Application

This project is learn how to containerize services in an application.

## Scenario

- You have a Multi Tier Application Stack
- Running on VMs
- Regular Deployment
- Continuous Changes

## Challenges

- High CapEx & OpEx
- Human Errors in deployment
- Not compatible with microservice architecture
- Resource Wastage
- Not portable, Env not in sync

## Solution

- Containers
- Consumes Low Resources
- Suitable for microservice design
- Deployments via Images
- Same container across all environments
- Reusable & Repeatable

## Tools

- Docker as the Runtime Environment
-  JAVA Stack VPROFILE Application Services

## Steps

- Gather all details from developers, i.e. know how the project has to be build and run, get the setup document, i.e. versions, packages, services running, how to build it and run it. Of course in this case we have everything through the source code.
- Steps to setup our [stack services](/vprofile-local-env/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.md)
- Find the right images from dockerhub
- Write Dockerfile to customize Images
- Write docker compose.yaml file to run multi containers
- Test it & Host Images on Dockerhub

## Containerization Project Diagram
![Project Diagram](Docker%20Workflow.png)
### Diagram Workflow
1. Fetch source code from the Git Repository
2. Write Docker files for services that needs customization ie. nginx, MySQL, and Tomcat. The base images will be pulled from dockerhub
3. Docker Build, to build images for them on the Docker Engine
4. Will use Docker Compose, will mention all the container with the images
5. Test the application and push the containers to dockerhub for storage if all goes well.

## Containerizing the [Vprofile Project](/vprofile-local-env/)

### Prerequisites

1. Dockerhub Account with an organization or not (optional)
   
   - To create an organization for collaboration, go to [organizations](https://hub.docker.com/orgs) > Create Organization > Business
     - Organization Details > Purchase
   - Benefits of organization
     - You can collaborate with team members
     - You have repositories named by organization not by account name
     - You can create teams
     - You can link HitHub or BitBucket accounts where images can be fetch image and build a container for you using its CICD
     - You can set default repository privacy
     - Notifications of build failures or successes to Slack or Email
  
    **This should be skipped on this project, not necessary.**

2. Vagrant VM with Docker Engine installed, refer to [Docker Engine Installation](https://github.com/mbadwa/devops-notes/blob/main/Docker-notes/docker-notes.md). Alternatively, a cloud VM.
   
3. Container images for MySQL, Memcached, RabbitMQ, Tomcat, and Nginx 

### Services

1. Memcached
   
   As per requirements it has to expose the port 11211. No other special [requirements](/vprofile-local-env/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.md). The official docker [image](https://hub.docker.com/_/memcached) runs on the same port, as per this [latest](https://github.com/docker-library/memcached/blob/f01103ba6999e5ae31c3e11f0d5bf9ee757aff44/1/debian/Dockerfile) tag.

2. MySQL
   
   According to the requirements MySQL container needs to have a specific username, password, db name, and also a db schema provided. As such these configurations will require a Dockerfile created from scratch using the [Dockerhub](https://hub.docker.com/_/mysql) as its base.
    

    **Note:** Always be aware of the [requirements](/vprofile-local-env/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.md) from the developers, the tags, the Docker compose creation, the actual commands to run. 

3. RabbitMQ

   This requires username and password, etc. In this case there is no need for special configurations. The [official image](https://hub.docker.com/_/rabbitmq) should suffice. No other special [requirements](/vprofile-local-env/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.md)

4. Tomcat
   
   This will require editing a tomcat image and put our artifact in it and customize the application.properties file accordingly as per our [requirements](/vprofile-local-env/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.md).

5. Nginx
   
   We need to run Nginx with the configuration mentioned as per [requirements](/vprofile-local-env/vagrant/Manual_provisioning_WinMacIntel/VprofileProjectSetupWindowsAndMacIntel.md). This can be done using a volume of by using a configuration file.

### Containerization Process

1. Log into your GitHub, fork the [source code](https://github.com/devopshydclub/vprofile-project/tree/containers), uncheck "Copy the vp-rem branch only" and hit Create fork
2. Clone the repo to your computer with VS Code and save the project within the folder where your Docker Engine's Vagrantfile is located
3. Switch to the branch Containers

        git branch
        git checkout containers
        git branch
4. Log into your [Dockerhub](https://hub.docker.com/) amd create repositories

   - Go to Repositories > Create repository
     - Namespace: your-name-space
     - Repository Name: "vprofileapp"
     - Visibility
       - Public
     - Hit Create

   - Create repository
     - Namespace: your-name-space
     - Repository Name: "vprofiledb"
     - Visibility
       - Public
     - Hit Create

   - Create repository
     - Namespace: your-name-space
     - Repository Name: "vprofileweb"
     - Visibility
       - Public
     - Hit Create 
5. Create a Tomcat Dockerfile

   - To create a Dockerfile, open your VS Code > Docker-files > app > [Dockerfile](/containerization/vproapp/Dockerfile) and paste the code.

6. Create MySQL Dockerfile

    - To create a Dockerfile, open Docker-files > db > [Dockerfile](/containerization/vproddb/Dockerfile) and paste the code.

7. Create Nginx Dockerfile and nginx config file
   
    - To create a Dockerfile and config file, open Docker-files > web > [Dockerfile](./web/Dockerfile) and the [nginxvproapp.conf](./web/nginxvproapp.conf)

8.  Create Docker Compose

    - To create Docker Compose file copy and paste the content from [docker-compose.yaml](./docker-compose/docker-compose.yml)

9.  Test the build in Docker Engine Vagrant VM

        vagrant reload
        vagrant ssh
        sudo -i
        clear
        cd /vagrant/vprofile-project
        ls
        vim docker-compose.yml
        docker compose build

10. Check the images and test running the containers

        docker images
        docker compose up d
        docker ps
11. Test the web app, copy your bridged ip or the ip in range 192.168.56.x and paste it in your browser

    1.  Log into the web 
        
            ip add show
            
            # Log in
            username: admin_vp
            password: admin_vp

    2.  Check on RabbitMQ
            
            Click on RabbitMQ

    3.  Check Memcached
            
            Click on All Users > click on any user

            # You'll see the message
            [ Data is From DB and Data Inserted In Cache !!]

            # Click on Back button and click on the same user again, the message cache is now updated
            [Data is From Cache]

12. Push the images to Dockerhub, log into your Dockerhub with your credentials

    1. Login and list images
            
            docker login
            docker images
    
    2. Push your images to Dockerhub
            
            docker push your-docker-acc/vprofileapp
            docker push your-docker-acc/vprofiledb 
            docker push your-docker-acc/vprofileweb 
    
    3. Remove the running Docker containers
            
            docker compose ps
            docker compose stop
            docker compose rm

        Note: Run the commands one by one, when you run *docker compose stop* it stops the containers, then you need to remove with *docker compose rm*. If you want to stop and remove at once run *docker compose down*    
    
    4. Verify that all containers are stopped and removed
    
            docker compose ps 
            docker compose ps -a
    
    5.  To clean all images

            docker system prune -a  

13. Commit your code to GitHub

        git branch
        git status
        git add .
        git commit -m "Updated Docker-files and Docker Compose file"
        git push origin containers
14. mm
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

- Steps to setup our stack services
- Find the right images from dockerhub
- Write Dockerfile to customize Images
- Write docker compose.yaml file to run multi containers
- Test it & Host Images on Dockerhub

## Diagram Workflow
1. Fetch source code from the Git Repository
2. Write Docker files for services that needs customization ie. nginx, MySQL, and Tomcat. The base images will be pulled from dockerhub
3. Docker Build, to build images for them on the Docker Engine
4. Will use Docker Compose, will mention all the container with the images
5. Test the application and push the containers to dockerhub for storage if all goes well.
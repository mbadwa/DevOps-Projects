# Deploying  a web app in a K8s

## Scenario

- Multi Tier Web Application Stack
- Containerized
- Tested
- Want to Host for Production
  

## Requirements

- High Availability
  - High availability so the containers do go down
  - High availability on the compute nodes
- Fault Tolerance
  - If something happens to the containers and they are not working properly, they should be able to auto heal
- Easy Scalability
  - Easy to scale the containers 
  - Easy to scale computer resource on which containers are running
- Platform Independent 
  - Agnostic to any Operating System
- Portable & Flexible
  - They should be agile, the containers can be run on local environment physical environment, cloud container runtime, cloud Vms. Also should be able to run in Dev, Test/QA and Prod environment, easily and conveniently

## Kubernetes Orchestration Tool 

Kubernetes is the best orchestration tool there is in the market, it's very mature and it's a rock solid platform to run containers for production.

## Technologies

- Java Vprofile Application Services
- Run it on Kubernetes Cluster for Production

## Steps

- Create a Kubernetes Cluster (kOps)
- Containerized apps (vprofile)
- Create EBS volume for DB Pod
- LABEL Node with zones names
  - This is to make sure the EBS volume is in the same zone as the Node where the DB Pod is located
- Write a Kubernetes Definitions file for these objects;
  - Deployment
  - Service
  - Secret
  - Volume
  
### 1. Create a K8s kOps Cluster

- The kOps kubernetes cluster instructions can be found [here](https://github.com/mbadwa/devops-notes/blob/main/Kubernetes-notes/kubernetes-notes.md), search for *Setup with Kops* keywords.
- Create a Kubernetes Repository in GitHub
- Clone repo into your code editor

### 2. Create an EBS for the DB pod in your kOps 

    $ aws ec2 create-volume --availability-zone=us-east-1a --size=3 --volume-type=gp2

Output

    {
    "AvailabilityZone": "us-east-1a",
    "CreateTime": "2024-09-14T17:46:26.000Z",
    "Encrypted": false,
    "Size": 3,
    "SnapshotId": "",
    "State": "creating",
    "VolumeId": "vol-045122c0ddbb94a0e",
    "Iops": 100,
    "Tags": [],
    "VolumeType": "gp2",
    "MultiAttachEnabled": false
    }

Take note of the ```VolumeId```

### 3. Create labels

  - You can check available labels

        $ kubectl get nodes --show-labels
  
  - List nodes to get a node name
  
        $ kubectl get nodes

    Output

        NAME                  STATUS   ROLES           AGE     VERSION
        i-00e53df3dfea44895   Ready    control-plane   8m32s   v1.26.15
        i-065109ea64c9db527   Ready    node            3m56s   v1.26.15
        i-0affd09a86e81dc7d   Ready    node            4m23s   v1.26.15
  
  - Describe the node to see the zone, we need a node in ```us-east-1a```
  
        $ kubectl describe node i-0affd09a86e81dc7d | grep us-east-1a
  
    Output

        failure-domain.beta.kubernetes.io/zone=us-east-1a
        topology.ebs.csi.aws.com/zone=us-east-1a
        topology.kubernetes.io/zone=us-east-1a
        ProviderID:                   aws:///us-east-1a/i-0affd09a86e81dc7d
   
  - Create a label for the node in ```us-east-1a```
  
        $ kubectl label nodes i-0affd09a86e81dc7d zone=us-east-1a

  - Create a label for node in ```us-east-1b```
  
        $ kubectl label nodes i-065109ea64c9db527 zone=us-east-1b

  - mm
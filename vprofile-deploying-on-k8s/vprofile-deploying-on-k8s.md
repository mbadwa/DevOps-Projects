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

### 4. Create a Kube Secret for Passwords

The secrets being created are for the passwords of the DBs found in properties file in source [code](https://github.com/devopshydclub/vprofile-project/blob/docker/src/main/resources/application.properties)

- Encode MySQL Password

      $ echo -n "vprodbpass" | base64

  Output

      dnByb2RicGFzcw==

- Encode RabbitMQ Password
  
      $ echo -n "guest" | base64

  Output

      Z3Vlc3Q=

- Create ```secret``` definitions [file](/vprofile-deploying-on-k8s/app-secret-yaml) named ```app-secret.yaml``` and commit it to the repo

      $ git add .
      $ git commit -m "secret"
      $ git push origin main 

- Test the definition file in kOps server

      $ clone https://github.com/mbadwa/kube-vproapp.git
      $ cd kube-app
      $ cat app-secret.yaml
      $ kubectl create -f app-secret.yaml

  Output 

      secret/app-secret created

  List secrets

      $ kubectl get secret

  Output

      mmmmm
  
  Describe secret

      $ kubectl describe secret

  Output

### 5. Create DB Deployment Definitions file

- Create a tag for the DB Volume before proceeding
  
  - Go to EC2 > Elastic Block Store > Volumes
    - Select *your-DB-volume* > Tags > Create Tag
      - Key: KubernetesCluster
      - Value: mbadwa.com
  - Hit Save

  NOTE: *the* ```Value``` *is your cluster name*


- Create a ```deployment``` [file](vprodbdep.yaml) named ```vprodbdep.yaml```
  
  NOTE: Line 40 creates a volume and then formats it to ext4, when Linux does that, it will create a folder names ```lost+found``` in ```/var/lib/mysql``` folder. When this happens, it inhibits MySQL service to start. Solution; create an init container found at line 45; ```initContainers``` to delete the ```lost+found``` folder first then subsequently create the runtime container.

- Commit and test the file
  
  Commit

      $ git add .
      $ git commit -m "Added vprodb deployment definitions file"
      $ git push origin main

  Test in kOps

      $ git pull
      $ kubectl create -f vprodbdep.yaml

  Output

      deployment.apps/vprodb created

  List deployments

      $ kubectl get deploy

  Output

      mmmm

  List pods

      kubectl get pod

  Output

      mmmmm

  Describe pod

      $ kubectl describe pod pod-name

  Output

      nmmmmm

  List Pods again

      $ kubectl get pod

### 6. Create DB Service Definition file 

Create a service with clusterIP because it's a DB not exposed to the outside world

- Create a Service ```clusterIP``` [file](db-CIP.yaml) definitions named ```db-CIP.yaml```
  
### 7. Create a Memcached Deployment & Service

Create a deployment definitions file for Memcached 

- Create a defs [file](mc-dep.yaml) named ```mc-dep.yaml``` 

Create a Service defs file for Memcached

- Create a defs [file](mc-CIP.yaml) named ```mc-CIP.yaml```

### 8. Create a RabbitMQ Deployment & Service

Create a deployment defs file for RabbitMQ

- Create a defs [file](rmq-dep.yaml) named ```rmq-dep.yaml```

Create a Service deployment defs 

- Create a defs [file](rmq-CIP.yaml) named ```rmq-CIP.yaml```

### 9. Create a Tomcat Deployment, Service and Init containers

Create a deployment defs file for Tomcat

- 
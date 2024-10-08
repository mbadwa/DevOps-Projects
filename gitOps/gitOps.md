# GitOps

Everything with code, GitOps is a subset of DevOps. A best strategy of modern DevOps. Simply put, it is `Everything in code & Git for code`, all changes should be managed by Git.

### Automation Problems

1. Automation and also manual changes
2. Drift in Infrastructure
3. No history of changes
4. Microservices complexity
5. No versioning of infra level changes

Git is a version control system, now how about extending it to manage operation changes. That's where GitOps comes in.

### Ingredients

- Code
  - CICD automation code
  - Infra automation code
- Git
  - versioning all the changes
  - Single place automation code tracking
  - Restricting users access to only git
- Tools 
  - Tool to read changes in git
  - Apply the differences at the infra level, the pipeline level and dev code level

In summary, for example you have cloud infra, e.g. VPC, RDS, Cache, EKS to host an app on it. This is the business logic part of it.

The code's version control is managed by **Git**, i.e. the App code, CICD Pipeline code (Jenkins), and cloud automation code (Terraform/Cloud Formation). This in practice should be different repositories  that are separate from each other. All these repos will be Git repositories, these changes are automated to a certain extent and the missing parts are done manually, this end the end brings up the challenges mentioned before. In an nutshell GitOps tries to restrict manual changes through applying policies in place. For example, no console access for the user, just programmatically accessing the cloud.

### Solution

All users will commit their respective teams code i.e. dev team, devops team, and ops team in Git, then use so called **GitOps Tools** listed below, they'll detect the changes at infra level, pipeline, or code level and then apply these changes accordingly.

- Github Actions
- GitLab
- ArgoCD
- Tekton
- Jenkins X

### Architecture of the Project

![alt](GitOps.png)

We will have two separate repositories on GitHub, one for Terraform code and the other for the application code. Both will have separate workflows which will detect and track the changes and trigger appropriate responses to apply the changes accordingly. 

We will fork two Git repos, set the SSH authentication with the repos. Use Visual Studio Code to write the workflows. The VS Code repos will be integrated with GitHub, the first will be creating Terraform Workflow by using GitHub Actions to implement any changes. 

**Terraform Workflow/Pipeline/CICD**

- First, it will fetch source code that has two branches, namely; `stage` and `main`
- When changes are made to the `staging` branch, the workflow will detect it, the Terraform code will be tested, basically it run two commands, `terraform validate` and `terraform plan` commands to check and test the code against AWS cloud and return the list of changes that will be applied before the actual application. The workflow will then break complete
- When the `staging branch` is validated successfully, it'll be merged to the main branch, this brach is always locked to apply any changes. For any changes to be applied, a `pull request` should be made by the engineer who was working on the code changes, the changes are then checked by the owner of the main branch who can approve or disapprove the `pull request` that will merge the staging branch with the main branch
- When the main branch detects the changes, the workflow will apply the changes to AWS infrastructure resources, ie. VPC and EKS

**Application Code Workflow/Pipeline/CICD**

This will have application code, a Dockerfile and Kubernetes definitions files, this is to apply any changes in the application.

- We will have a workflow that'll fetch the code, build the code, test the code and deploy the code
- We will use Maven CheckStyle and Sonar Code Analysis CLI that will test the source code and validate it with Sonarcloud Quality Gates
- If everything checks out fine, it'll build docker images and upload them to Amazon ECR
- We will use Helm Charts, these are bundles of Kubernetes Definition file, it 'll also have a variable that mentions the `tag` and `image` names, basically, from where to fetch the image and what image to fetch
- This information will be parsed in the workflow automatically, the tag info will be passed to the Helm Charts, and the Helm Charts will executed on the EKS cluster, the Kubernetes cluster will detect the change of the image tag and it will fetch from ECR and run the application
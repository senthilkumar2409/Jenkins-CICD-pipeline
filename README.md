# Jenkins CI/CD pipeline to build, test and deploy an application in aws eks:

### Pre-requisite:

1. Setup a Jenkins server on aws ec2 instance.
2. Setup a sonarqube server as a docker container in ec2 instance
3. Install docker in jenkins server
4. Provision a aws eks cluster in aws cloud (Used terraform to provision a eks cluster - https://github.com/senthilkumar2409/terraform-cicd-eks terraform configuration files exist in this repo)

## 1. Jenkins pipeline setup:

### Install Plugins
   Install plugins to integrate Jenkins with GitHub, Maven, and EC2. Go to Manage Jenkins, and select Manage plugins. Under available plugins search for the below plugins and Install without restart
   1. Git
   2. maven
   3. docker
   4. aws ecr - It generates the docker authentication token from aws credentials to access ECR repository
   5. Sonarqube Scanner

   ![image](https://github.com/user-attachments/assets/751d7010-cc41-429b-a266-a9f19981077e)


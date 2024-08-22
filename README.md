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

### Configure maven and sonarqube:
   Go to Manage Jenkins, select tool configuration, and scroll down to add Maven and Sonarqube path, steps as shown below.
    
   ![image](https://github.com/user-attachments/assets/c30afc8c-03cc-490a-b5ca-b86c4a3e3e90)
   ![image](https://github.com/user-attachments/assets/079ccfcd-dde6-4834-8461-19ed54451f41)
   
### Setup Sonarqube server and generate a token:
   Go to Manage Jenkins, system page, and scroll down to add sonarqube server,
   * Add SonarQube: Click Add SonarQube.
     
      1.Name: Provide a name for your SonarQube server, e.g., sonarqube-server.
     
      2.Server URL: Enter the URL where your SonarQube server is accessible, e.g., http://<server-url>:9000.
   
   * Generate a sonarqube token.

      1.In the My Account page, navigate to the Security tab.
     
      2.Under the Tokens section, you’ll see an option to Generate Tokens.
     
      3.Enter a name for your token in the Token Name field. This name is for your reference and helps you identify the token later (e.g., JenkinsCI) and Click the Generate button.






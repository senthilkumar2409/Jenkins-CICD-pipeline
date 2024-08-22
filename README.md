# Jenkins CI/CD pipeline to build, test and deploy an application in aws eks:

### Pre-requisite:

1. Setup a Jenkins server on aws ec2 instance.
2. Setup a sonarqube server as a docker container in ec2 instance
3. Install docker in jenkins server
4. Provision a aws eks cluster in aws cloud (Used terraform to proviosion a eks cluster - https://github.com/senthilkumar2409/terraform-cicd-eks)

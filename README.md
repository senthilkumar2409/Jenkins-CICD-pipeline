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

### Credentials:
   Store required credentials for jenkins to access a necessary tools or application like git, soanrqube server and aws(aws credentials to provide access to ecr repository and eks cluster) credentials 

## 2. Jeninsfile:

```groovy
pipeline {
    agent any
    
    tools {
        maven 'maven3'
        //Docker 'docker'// This should match the name of the Maven tool in the Global Tool Configuration
    }

    environment{
        DOCKER_IMAGE = 'shopping_cart'
        DOC_ECR_REPO = '975049977826.dkr.ecr.us-east-1.amazonaws.com'
    }

    stages {
        stage('git checkout') {
            steps {
                git credentialsId: 'cred', url: 'https://github.com/senthilkumar2409/shopping_cart.git'
            }
        }
        stage('maven build') {
            steps {
                sh 'mvn package install -Dmaven.test.skip=true' 
            }
        }
        stage('sonarqube analysis') {
            environment{
                    SonarScanner = tool 'sonar-scanner6'
                }
            steps {
                withSonarQubeEnv("sonar-scanner-server") {
    // some block
                sh ''' $SonarScanner/bin/sonar-scanner -Dsonar.projectName=shopping_cart \
                -Dsonar.projectKey=shopping_cart \
                -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('docker build') {
            steps {
                sh 'docker build -f docker/Dockerfile -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .' 
            }
        }
        stage('docker image scan') {
            steps {
                sh 'trivy -f table -o scan_report.txt image ${DOCKER_IMAGE}:latest' 
            }
        }
        stage('Login to ecr registry and docker push') {
            steps {
                script{
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws_cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
    // some block. 
                   sh ' aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 975049977826.dkr.ecr.us-east-1.amazonaws.com'
                   sh ' docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOC_ECR_REPO}/${DOCKER_IMAGE}:${BUILD_NUMBER}'
                   sh ' docker push ${DOC_ECR_REPO}/${DOCKER_IMAGE}:${BUILD_NUMBER}'
                    
                    }
                }
            }
        }
        stage('Update Kube manifest file') {
            steps {
                script{
                    sh '''
                        export DOCKER_REPO=${DOC_ECR_REPO}
                        export IMAGE_NAME=${DOCKER_IMAGE}
                        export IMAGE_TAG=${BUILD_NUMBER}
                        envsubst < deploymentservice.yml > deployment.yaml
                    '''

                }
            }
        }
        stage('Deployment on EKS') {
            steps {
                script{
                  sh '''
                     kubectl apply -f deployment.yaml -n kube-system
                     '''
                }
            }
        }

    }
}
```






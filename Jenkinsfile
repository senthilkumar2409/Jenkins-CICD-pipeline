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
                     aws eks --region us-east-1 update-kubeconfig --name terraform-prod
                     kubectl apply -f deployment.yaml -n kube-system
                     '''
                }
            }
        }

    }
}

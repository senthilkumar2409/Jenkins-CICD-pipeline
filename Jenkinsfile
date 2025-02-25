pipeline {

    agent any
     
    tools {
        maven 'maven3.6' 
        //Docker 'docker'// This should match the name of the Maven tool in the Global Tool Configuration
    }
    
    environment{
        
        DOCKER_IMAGE = 'shopping_cart'
        DOC_ECR_REPO = '975049977826.dkr.ecr.us-east-1.amazonaws.com'
    } 

    stages {
        stage('maven build') {
            steps {
                sh 'mvn package install -Dmaven.test.skip=true' 
            }
        }
        stage('docker build') {
            steps {
                sh 'docker build -f docker/Dockerfile -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .' 
            }
        }
        stage('docker image scan') {
            steps {
                sh 'trivy -f table -o scan_report.txt image ${DOCKER_IMAGE}:${BUILD_NUMBER}' 
            }
        }
        stage('Login to ecr registry and docker push') {
            steps {
                script{
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
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
         stage('Push to github') {
             steps {
                 script{
                    withCredentials([string(credentialsId: 'GITHUB ARGOCD', variable: 'GITHUB')]) {
    // some block

                      sh '''
                        git init
                        git config user.email "senthil24091999@gmail.com"
                        git config user.name "senthilkumar2409"
                        git pull origin main
                        git add deployment.yaml
                        git commit -m " deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB}@github.com/senthilkumar2409/argocd_repo HEAD:main
                    '''
                    }
                 }
             }
         }
        stage('workspace cleanup') {
            steps {
                script{
                  cleanWs()
                }
            }
        }
    }
}
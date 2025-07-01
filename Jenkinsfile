pipeline {

    agent any
     
    tools {
        maven 'maven3.6' 
        //Docker 'docker'// This should match the name of the Maven tool in the Global Tool Configuration
    }
    
    environment{
        
        IMAGE_NAME = 'shopping_cart'
        IMAGE_TAG = "{BUILD_NUMBER}"
        DOC_ECR_REPO = '975049977826.dkr.ecr.us-east-1.amazonaws.com'
    } 

    stages {
        stage('build and test') {
            steps {
                sh 'mvn clean package' 
            }
        }
        stage('SonarQube Analysis') {
          steps {
               withSonarQubeEnv('sonarqube') {
                sh '''
                     mvn sonar:sonar \
                            -Dsonar.projectKey=voting-service \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_AUTH_TOKEN}
                    '''
                }
            }
        }
        stage('Quality Gates') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy Artifact to AWS CodeArtifact') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', 
                                  credentialsId: 'aws-codeartifact-creds', 
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh '''
                        # Authenticate to CodeArtifact
                        aws codeartifact login \
                            --tool maven \
                            --domain ${CODEARTIFACT_DOMAIN} \
                            --repository ${CODEARTIFACT_REPOSITORY} \
                            --region ${AWS_REGION}
                        # Deploy artifact
                        mvn deploy -DskipTests
                    '''
                }
            }
        }
        stage('docker build') {
            steps {
                sh 'docker build -f docker/Dockerfile -t ${IMAGE_NAME}:${IMAGE_TAG} .' 
            }
        }
        stage('docker image scan') {
            steps {
                sh "trivy image --severity HIGH,CRITICAL --exit-code=1 --format table -o trivy-report.txt ${IMAGE_NAME}:${IMAGE_TAG}"
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
        stage('update helm') {
            steps {
                script {
                       sh '''
                        
                        sed -i "s/tag: \".*\"/tag: \"$BUILD_NUMBER\"/" deployment/shoppingkart/values.yaml 
                        cat ${FILE_PATH}

                        '''
                }
            }
                
        }
        stage('push to github') {
            steps {
          withCredentials([string(credentialsId: 'github-argocd', variable: 'Github')]) {
             script {

                sh 'git config --global user.name "senthilkumar2409"'
                sh 'git config --global user.email "senthil24091999@gmail.com"'
                sh 'git add ${FILE_PATH}'
                sh 'git commit -m "updated the helm values.yaml file with ${BUILD_NUMBER}"'
                sh 'git push https://$Github@github.com/senthilkumar2409/argocd_repo.git HEAD:master' 
                 }
            }
        }
    }
        
        post {
          success {
              slackSend(color: 'good', message: "Pipeline Successfull: ${env.JOB_NAME} ${env.BUILD_NUMBER} ${env.BUILD_URL}") 
          }
          failure {
              slackSend(color: 'danger', message: "Pipeline Failed: ${env.JOB_NAME} ${env.BUILD_NUMBER} ${env.BUILD_URL}") 
          }
          aborted {
              slackSend(color: 'warning', message: "Pipeline Aborted: ${env.JOB_NAME} ${env.BUILD_NUMBER} ${env.BUILD_URL}")
          }
}

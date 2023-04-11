def remote = [:]
    remote.name = 'Docker-server'
    remote.host = '54.161.108.229'
    remote.user = 'ubuntu'
    remote.password = 'December2023#'
    remote.allowAnyHosts = true

pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"
        AWS_ACCOUNT_ID="111393898725"
        AWS_DEFAULT_REGION="us-east-1"
        IMAGE_REPO_NAME="dec2-webapp"
        IMAGE_TAG= "${env.BUILD_ID}"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
    stages {
          stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: '', url: 'https://github.com/evarestigwe/sample-web-app.git'
            }
        }

          stage('Logging into AWS ECR') {
                     environment {
                        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                         
                   }
                     steps {
                       script{
             
                         sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                }
                 
            }
        }
         
          stage('Building docker image') {
            
            steps{
              script {
                dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
        
          stage('Pushing to ECR') {
             environment {
                        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
                        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
                         
                   }
              steps{  
                script {
                    withAWS(role: "dec-role", roleAccount: '111393898725') {
                    sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                    sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
                    sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
         }
        }
      }
 }
        stage('Deploy to Decker-SErver Via SSH') {
          steps{
      sshCommand remote: remote, command: "ls -lrt"
      sshCommand remote: remote, command: "aws --profile dec-user ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 111393898725.dkr.ecr.us-east-1.amazonaws.com"
      sshCommand remote: remote, command: "docker pull 111393898725.dkr.ecr.us-east-1.amazonaws.com/dec2-webapp:9"
      sshCommand remote: remote, command: "docker run -d -p 80:80 --name webapp 111393898725.dkr.ecr.us-east-1.amazonaws.com/dec2-webapp:9"
      }
      }  


    }
}

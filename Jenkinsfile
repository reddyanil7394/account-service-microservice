 pipeline {
    tools { 
        maven "Maven3"
    }
    
    environment {
        registry = "353258999787.dkr.ecr.us-east-1.amazonaws.com/account_service_microservice"
    }
    
    agent any
  
    stages {
        stage('checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/anilreddy7394/account-service-microservice.git']])
            }
        }
        
        stage('Build Jar') {
            steps {
                sh "mvn clean install"
            }
        }
     
        stage('SonarQube Analysis') {
            steps {
               sh "mvn clean verify sonar:sonar \
               -Dsonar.projectKey=jenkins-pipeline \
               -Dsonar.host.url=http://52.86.114.38:9000 \
               -Dsonar.login=sqp_7298ccece2bcd05561a94ac085c8eb645562d4b5"
           }
       }
        
        stage('Build image') {
            steps {
                script {
                    docker.build registry
                }
            }
        }
        
        stage('Push into ECR') {
            steps {
                sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 353258999787.dkr.ecr.us-east-1.amazonaws.com"
                sh "docker push 353258999787.dkr.ecr.us-east-1.amazonaws.com/account_service_microservice:latest"
            }
        }
        
        stage('K8S Deploy to production') {
            when {
                branch 'master'
            }
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', serverUrl: '') {
                sh "kubectl apply -f eks-account-service.yaml"
            }
          }
          post{
              success{
                  echo "Successfully deployed to production"
              }
              failure{
                  echo "Failed deploying to production"
              }
           }
       }
       stage('K8S Deploy to staging') {
            when {
                branch 'devlopment'
            }
            steps {
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', serverUrl: '') {
                sh "kubectl apply -f eks-account-service-staging.yaml"
            }
          }
          post{
              success{
                  echo "Successfully deployed to staging"
              }
              failure{
                  echo "Failed deploying to staging"
              }
           }
       }
       
    }
}

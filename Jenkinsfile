pipeline {
    agent any
    environment{
        GIT_REVISION_NUMBER= """${sh(
                returnStdout: true,
                script: 'git rev-parse HEAD'
            )}"""
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '**']], extensions: [], userRemoteConfigs: [[credentialsId: 'GitHub', url: 'https://github.com/abhisheknanarkar24/python-poc.git']]])
            }
        }

        stage('SonarQube Analysis') {
          def scannerHome = tool 'sonarqube';
          withSonarQubeEnv() {
            sh "${scannerHome}/bin/sonar-scanner"
            }
            }
        
        stage('Build Docker Image') {
            steps {
               sh 'docker build -t $GIT_REVISION_NUMBER .'
               
            }
        }
        stage('push docker image to ECR') {
            steps {
                withAWS(credentials: 'abhishek_aws', endpointUrl: 'https://294426219574.signin.aws.amazon.com/', region: 'us-east-1') {
                    sh '''/usr/local/bin/aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 294426219574.dkr.ecr.us-east-1.amazonaws.com
                    docker tag $GIT_REVISION_NUMBER 294426219574.dkr.ecr.us-east-1.amazonaws.com/app:$GIT_REVISION_NUMBER
                    docker push 294426219574.dkr.ecr.us-east-1.amazonaws.com/app:$GIT_REVISION_NUMBER'''
                    }
             
            }
        }
        stage('Lambda Deployment') {
            steps {
               withAWS(credentials: 'abhishek_aws', endpointUrl: 'https://294426219574.signin.aws.amazon.com/', region: 'us-east-1') {
               sh '/usr/local/bin/aws lambda update-function-code --region us-east-1 --function-name app-poc  --image-uri 294426219574.dkr.ecr.us-east-1.amazonaws.com/app:$GIT_REVISION_NUMBER'
                            }
            }
            
    }}}